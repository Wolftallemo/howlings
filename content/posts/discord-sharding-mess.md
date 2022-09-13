---
title: "Discord's Sharding is a Mess"
date: 2022-09-12T18:19:39-04:00
draft: false
---

There are many things to consider when creating a Discord bot. Sooner or later, if your bot is
public, you will eventually need to consider sharding. In fact, you get disconnected and cannot
reconnect without sharding if the bot is in 2500+ servers.

So how do you do this? The basics are relatively simple. As per
[Discord's Developer Docs](https://discord.com/developers/docs/topics/gateway#sharding),
with each shard you send an array with the format `[shard_id, num_shards]`. `shard_id` is exactly
what it implies, the ID number of the shard. But this is where the simplicity ends.

There are some other things ranging from mildly surprising to outright insane. Multiple
connections using the same `[shard_id, num_shards]` is allowed. The documentation isn't exactly
explicit about what happens when you do this, it mentions load balancing and zero-downtime updates,
but it's all mashed as one sentence. So when you spawn multiple sessions with the same
`[shard_id, num_shards]`, all of those shards receive the same events, so you must dedupe them.
This would usually mean setting a property telling the older shard to not process events. Alas,
this behavior is unsuitable for any form of load balancing.

Another thing about sharding, that some people who are new to bot development might not be aware
about, is that direct messages are always sent to shard 0. If you're familiar with how sharding is
handled on Discord's end, this isn't surprising, as direct message channels do not have a guild.
In many cases this isn't a problem, but if your bot receives a lot of direct messages, that means
all of them are being sent to one shard and there's nothing you can do about it.

If you want to scale up or down the number of shards your bot has, you can't just disconnect shards
or connect new ones. You have to disconnect every shard and send a new identify payload for each
one, although you can keep the old ones connected until the new shards are ready for a
minuscule-to-none downtime re-shard. You might end up processing duplicate events doing this,
however.

A somewhat strange thing you can do as well is specify different `num_shard` values for shards.
According to Discord, this can be done for load balancing purposes, as `num_shards` is only used
for routing. But how do you actually handle load balancing this way?

Well... Discord doesn't actually say how to do this, and a search in the Discord Developers server
doesn't turn up any results. Because of the first point I made above, any shard that matches their
calculation formula will receive events from a specific guild. This means that there may be some
overlap of shards, resulting in duplicate events. Alternatively, this could also mean that some
guilds don't have a matching shard at all, so the bot would appear offline in those guilds. So much
for load balancing...

Alas, my wishlist for Discord is for them to:

- Not send all DMs to shard 0: While direct message channels aren't in a guild, we could in theory
use the non-bot user's snowflake. As bots cannot be added to a group dm, I don't foresee any issues
managing DMs this way.
- Allow changing `num_shards` dynamically for all shards: This would most likely be a new gateway
command. It would allow dynamically scaling up or down with basically zero downtime (I say
basically, because guilds on the last shard[s] might have some very short downtime) without
re-identifying every shard.
- Clarify and explain how differing `num_shards` values can be used for load balancing: It's
strange that they mention such a use case, but they don't give an explainer on how to do it. It
would be very helpful especially as there isn't any obvious way to do it that is guaranteed to not
result in the same event being sent to multiple shards or some guilds not having a shard at all.

I suppose that I could spawn a separate process to manage gateway traffic, but this doesn't help
for direct messages (see my second point) or cases where you have a high number of large guilds
clustered on a few shards. Re-sharding is possible, but this isn't a guaranteed solution, since
it's always possible that you could worsen the issue.
