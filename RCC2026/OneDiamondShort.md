# One Diamond Short

RowdyCyberCon CTF 2026 · Game Hacking · Solved

## Challenge

Challenge based in a Minecraft clone with the objective to collect 10 diamonds. The catch is that only 9 diamonds are obtainable anywhere in the world, so the win condition can't be met by playing the game as intended.

## Recon

Running `strings` on the binary surfaced the win text and a file-write routine, with the flag format `RCC{%s}` written to `README.txt` on a win. That confirmed two things: the flag is generated locally on win, and the only thing standing between me and it is the diamond counter.

The diamond count is stored as a 4-byte integer that the game recomputes from inventory state. Because crafting can't create a 10th diamond, the intended path is to manipulate the counter in memory rather than find a missing diamond.

## Solution

Using Cheat Engine:

1. Attach to `OneDiamondShort.exe`.
2. Run a known-value scan on the current diamond count (4-byte integer).
3. Change the count in-game by picking up or dropping a diamond, then run **Next Scan** on the new value.
4. Repeat until only a few candidate addresses remain.
5. Set the value-9 addresses to **10** and freeze them, so the game's recompute can't revert the value back.

With the counter locked at 10, the win condition triggers and the game writes the flag to `README.txt`.



