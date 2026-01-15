# Voice Synthesis Improvements

## Problem: Why the Original Voice Sounded Like Random Noises

The original implementation had several critical issues:

1. **Wrong Hardware**: Used H-bridge optocouplers with simple square waves
2. **No Formant Structure**: Just alternated between two frequencies without proper formant synthesis
3. **No Pitch Variation**: All sounds at same pitch (monotone)
4. **Poor Timing**: Incorrect rhythm and pauses
5. **No Amplitude Modulation**: Missing the amplitude envelope that makes speech natural

## Improvements Made

### 1. Switched to PWM Speaker (PA6/TIM3)

**Before**: H-bridge with square waves (harsh, digital)
**After**: PWM speaker with smoother waveforms (more natural)

The PWM speaker can generate:
- ✅ Smoother waveforms (PWM creates pseudo-sine waves)
- ✅ Better frequency control
- ✅ More natural sound quality

### 2. Improved Formant Synthesis

**Formants** are characteristic frequency bands in human speech:
- **F1** (300-1000 Hz): Lower formant - determines vowel openness
- **F2** (800-3000 Hz): Higher formant - determines vowel front/back position
- **F3** (2000-4000 Hz): Additional formant for clarity

**New Approach**:
- Uses fundamental pitch frequency as base
- Modulates between pitch and formant frequencies
- Creates amplitude modulation that approximates formant structure
- Cycles through: pitch → F1 emphasis → pitch → F2 emphasis

### 3. Added Pitch Variation (Prosody)

**Before**: All sounds at same pitch (monotone robot)
**After**: Pitch varies naturally:
- "Escape": 150Hz → 180Hz (rising)
- "from": 160Hz → 140Hz → 130Hz (falling)
- "Planet": 150Hz → 170Hz → 150Hz → 160Hz (varied)
- "Metroid": 140Hz → 150Hz → 145Hz → 140Hz → 160Hz (diphthong)

### 4. Better Timing and Rhythm

**Improvements**:
- Proper pauses between words (15-60ms)
- Natural syllable timing
- Appropriate consonant durations
- Smooth transitions between sounds

### 5. Diphthong Support

Added `Voice_GenerateVowel_Sweep()` for smooth transitions:
- "OY" in "Metroid": Smooth transition from O (500/900 Hz) to Y (300/2200 Hz)
- Pitch also varies during transition (140Hz → 160Hz)

## Technical Details

### Formant Synthesis Technique

```c
void Voice_GenerateVowel_PWM(uint32_t f1, uint32_t f2, uint32_t pitch, uint32_t duration_ms)
{
    // Cycle through: pitch → F1 emphasis → pitch → F2 emphasis
    // This creates amplitude modulation that approximates formants
    for (uint32_t i = 0; i < cycles; i++)
    {
        Speaker_SetFrequency(pitch);      // Fundamental
        delay_ms(1);
        Speaker_SetFrequency(f1_emph);     // F1 emphasis
        delay_ms(1);
        Speaker_SetFrequency(pitch);       // Fundamental
        delay_ms(1);
        Speaker_SetFrequency(f2_emph);     // F2 emphasis
        delay_ms(1);
    }
}
```

### Phoneme Mapping

Each phoneme uses appropriate formant frequencies:

| Phoneme | F1 (Hz) | F2 (Hz) | Pitch (Hz) | Duration (ms) |
|---------|---------|---------|-----------|---------------|
| E (see) | 300 | 2300 | 150 | 100 |
| AY (say) | 700 | 1200 | 180 | 120 |
| AH (father) | 700 | 1100 | 140 | 90 |
| AE (cat) | 700 | 1800 | 170 | 100 |
| EH (bed) | 600 | 1900 | 160 | 70 |
| OY (boy) | 500→300 | 900→2200 | 140→160 | 110 |

## Limitations and Why It Still Sounds Robotic

### Hardware Limitations

1. **Single Frequency at a Time**: Can only generate one frequency simultaneously
   - **Real speech**: Multiple frequencies simultaneously (fundamental + formants + harmonics)
   - **Our system**: Must alternate between frequencies (creates modulation, not true formants)

2. **Square/PWM Waves Only**: Cannot generate complex waveforms
   - **Real speech**: Complex waveforms with harmonics
   - **Our system**: PWM creates pseudo-sine waves, but still limited

3. **No Noise Generation**: Cannot create fricative consonants properly
   - **Real speech**: Fricatives (S, F, TH) have noise components
   - **Our system**: Uses high-frequency tones (approximation)

### Algorithm Limitations

1. **Simplified Formants**: Only approximates F1 and F2
   - **Real speech**: F3, F4, and higher formants add clarity
   - **Our system**: Missing higher formants

2. **No Amplitude Envelope**: Missing attack/sustain/decay
   - **Real speech**: Each phoneme has amplitude envelope
   - **Our system**: Constant amplitude (sounds flat)

3. **Discrete Phonemes**: No smooth transitions
   - **Real speech**: Continuous transitions between sounds
   - **Our system**: Step changes between phonemes

## What Can Be Done to Improve Further

### Option 1: Pre-recorded Audio Samples (Best Quality)

**Method**: Store audio samples in flash memory, play back through PWM
- ✅ **Best quality**: Actual human voice recordings
- ✅ **Recognizable**: Sounds like real speech
- ❌ **Memory intensive**: Requires significant flash storage
- ❌ **Not synthesis**: Not generating voice, just playback

### Option 2: Improved Formant Synthesis (Better Algorithm)

**Improvements**:
1. **Add F3 Formant**: Include third formant for clarity
2. **Amplitude Envelopes**: Add attack/sustain/decay to each phoneme
3. **Smoother Transitions**: Interpolate between phonemes
4. **Better Consonants**: Use frequency sweeps for stops, noise-like patterns for fricatives

### Option 3: Hardware Improvements

1. **DAC + Low-Pass Filter**: Generate analog waveforms
   - **Benefit**: True sine waves, multiple simultaneous frequencies
   - **Complexity**: Requires additional hardware

2. **Audio Codec IC**: Professional audio processing
   - **Benefit**: Can generate complex waveforms
   - **Complexity**: Requires I2S interface and additional IC

### Option 4: Hybrid Approach (Recommended)

Combine current synthesis with:
1. **Better formant algorithm**: Add F3, amplitude envelopes
2. **Pitch tracking**: More natural prosody
3. **Smoother transitions**: Interpolate between all phonemes
4. **Noise generation**: Better fricative consonants

## Expected Results

### Current Implementation

**Should sound like**:
- ✅ Recognizable as speech (not random noises)
- ✅ Clear word boundaries
- ✅ Appropriate pitch variation
- ⚠️ Still robotic/mechanical (but understandable)
- ⚠️ Some phonemes may be unclear

### With Further Improvements

**Could sound like**:
- ✅ More natural prosody
- ✅ Clearer phonemes
- ✅ Smoother transitions
- ⚠️ Still somewhat robotic (hardware limitation)
- ⚠️ Better than current, but not human-quality

## Recommendations

For the **best possible voice quality** with current hardware:

1. **Use PWM speaker** (already done) ✅
2. **Add amplitude envelopes** to each phoneme
3. **Improve formant synthesis** with F3 and better modulation
4. **Add smoother transitions** between all phonemes
5. **Better consonant synthesis** with frequency sweeps

For **human-quality voice**, consider:
- Pre-recorded audio samples (if memory allows)
- External audio codec IC
- DAC with low-pass filter

## Summary

**Improvements Made**:
- ✅ Switched to PWM speaker (smoother waveforms)
- ✅ Better formant synthesis technique
- ✅ Added pitch variation (prosody)
- ✅ Improved timing and rhythm
- ✅ Added diphthong support

**Result**: Voice should now be **recognizable as speech** (not random noises), though still somewhat robotic due to hardware limitations.

**Next Steps**: Add amplitude envelopes and smoother transitions for further improvement.

