# Highrise Trickshot

 RowdyCyberCon CTF 2026 · Web · Solved

## Challenge

A trickshot shooter game with the goal being landing a trickshot worth 10,000,000,000 points

## How scoring works

The key realization is that the score is not computed in the browser. Firing sends the entire jump to the server as an input log, and the server replays it through its own physics engine:

- POST /api/session — returns the bot position and a nonce.
- POST /api/takeoff — sends the nonce, jump, config, and takeoff state.
- POST /api/trickshot — sends the nonce, jump, frames, end, and player.

Each frame is [dt, [events], rawYaw, pitch, keyMask], and the final score is 100 × (product of all multipliers).

## Flaw:

The server replay trusts the client-supplied config, and the base config ships with MAX_TURN_RATE: 1e7. Spins multiply the score by (1 + spins), and that turn-rate cap allows roughly 27,778 full spins per second. Takeoff speed, takeoff position, and config values are nominally validated, but the base config already permits `1e7` so no tampering is even required to get an absurd spin budget.

The second insight is fire control. After spinning, yaw becomes a huge number and the per-frame turn clamp cant snap it back to aim. Spinning in exact whole revolutions per frame (K×2π, just under the cap) keeps `yaw mod 2π` constant, so the crosshair stays locked on the dummy for the entire fall.



## Solution:

Take off straight above the dummy at the ceiling (by+12, vel.y=7), fall straight down, and during the fall:

1. Spin at the rate cap in whole revolutions: ×38,851. Both directions with Reverse 360: ×1.5.
2. Desert Eagle, 5 straight-down tags: (1 + 3×5) = ×16. A sniper finish is blocked — the sniper shares the Eagles 1.1 s fire delay, so only about one tag fits.
3. Throwing knife finish: ×5. The knife ignores the fire delay, so tag spam is unrestricted.
4. Swap back to sniper just before the throw: weapon swap ×2 and canswap ×1.5.
5. Free multipliers from the vertical drop: mid-air ×2, airtime ×1.71, headshot ×1.5, "off the crane" ×1.5, kamikaze ×1.5, flick ×1.3.

**Final score: 23,972,740,845** well past the 10,000,000,000 target. The flag comes back in the flag field of the /api/trickshot response.


## Issues encountered:

- end must match the replay s final position, or the response is Killcam desync.
- No frames may follow the kill, the knife throw has to be the final frame.

## Takeaway

"Server-authoritative" isnt the same as secure. The server did replay the physics, but it replayed them using config the client supplied, which is just client trust wearing a disguise. The whole solve was driven through the backend API with a small Python (urllib) harness treating the servers responses as the oracle, never touching the intended front end.
