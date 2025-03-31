# Release notes

## Mar. 31, 2025

### Update

- Update the product number of CDD-OPT mezzanine card.
    - CDD-OPT v3 (GN-2436-1) is the latest.

## Mar. 5, 2025

### New + Bugfix

- Bugfix for previous version
    - All firmware released on Jan. 6 have critical bugs.
    - So far the function to insert throttling start/end words in the input throttling type-2 unit has not been working. Now that the feature is enabled, a word is generated with input throttling type-2 start/end data types to indicate that the throttling feature is working.
- New firmware
    - Str-LRTDC v2.8
    - Str-HRTDC Base v2.7
    - Mezzanine Str-HRTDC v2.7
    - Mikumari Clock Hub v2.7
    - Mikumari Clock Root v2.7
        - The Mikumari Clock Root v2.7 will be the final version. It will not be updated any further.

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
