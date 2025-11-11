+++
author = "lwcas"
title = "packets in online games"
date = "2025-11-10"
description = "understanding how data travels in online multiplayer games"
tags = [
    "networking",
    "multiplayer",
    "games",
]
+++

# what are packets in multiplayer games?

in multiplayer games, packets are like the envelopes or messages that carry information between players' devices and the game server. they ensure that actions like moving your character, shooting, or chatting are synchronized across all players in real-time. without packets, online gaming would be impossible—everyone would be playing in their own isolated world.

imagine you're playing a team-based game with friends:
- **server:** the central hub that knows everything happening in the game world.
- **clients:** each player's device, sending updates to the server and receiving updates from others.
- **packets:** the data bundles that travel over the internet, containing things like "player x moved to position (10, 20)" or "player y fired a bullet."

packets make sure everyone sees the same game state, even with network delays or losses. but if packets get lost or arrive out of order, it can cause glitches like rubber-banding or desyncs.

## how packets work in practice

packets are sent using network protocols, mainly tcp and udp:
- **tcp (transmission control protocol):** reliable but slower. it guarantees packets arrive in order and retransmits lost ones. used for important data like login info or inventory changes.
- **udp (user datagram protocol):** fast but unreliable. packets might get lost, but no waiting for retransmission. used for real-time actions like movement updates.

in games, a mix is often used. for example, movement might use udp for speed, while chat messages use tcp for reliability.

simple packet example (conceptual, not actual code):
```rust
// imagine sending a position update
let position_packet = Packet {
    player_id: 1,
    x: 100.0,
    y: 200.0,
    timestamp: get_current_time(),
};
send_to_server(position_packet);
```

on the server, it processes and broadcasts to other players:
```rust
// server receives and relays
if let Ok(packet) = receive_from_client() {
    broadcast_to_all_clients(packet);
}
```

## why packet handling matters

poor packet management can ruin the experience:
- **latency (ping):** time for a packet to go to the server and back. high latency causes delays in actions.
- **packet loss:** some packets don't arrive. games interpolate missing data to smooth it out.
- **jitter:** inconsistent delays. causes stuttering.
- **security:** packets can be spoofed or tampered with, so games use encryption and validation.

these issues are why games have "netcode" – the system handling network communication. good netcode predicts movements and corrects errors to keep the game fair and fun.

## extras

**dead reckoning:** predicting player positions based on last known data to hide latency. like assuming a car keeps moving straight until new info arrives.

**rollback netcode:** rewinding time to correct mistakes from lost packets, used in fighting games for precise timing.