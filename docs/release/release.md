# Release notes

## Jan. 6, 2025

### New + Improved

- New function of LACCP distributing frame flags
    - LACCP can distribute 2-bits frame flags. The flags are embedded into the flags regions in the heartbeat frame delimiter word.
    - The users can embed external condition such as gates into the heartbeat frame.
- Gated scaler
    - Two scaler units gated by the frame flags are added. Totally, 3 scaler units exist in firmware.
    - Gated scaler counts up while the frame flag takes 1.
- New firmware
    - Str-LRTDC v2.6
    - Str-HRTDC Base v2.5
    - Mezzanine Str-HRTDC v2.5
    - Mikumari Clock Hub v2.6
    - Mikumari Clock Root v2.6
    - The Mikumari Clock Root firmware will be deprecated in the next update because the Mikumari Clock Hub can do the same thing.

## Jun. 4, 2024

### New

- De facto release version
    - Str-LRTDC v2.5
    - Str-HRTDC Base v2.4
    - Mezzanine Str-HRTDC v2.4
    - Mikumari Clock Root v2.5
    - Mikumari Clock Hub v2.5

