# WHY THIS FORK?

SoX Resampler Library **by default produces non-deterministic output** (i.e.
each time you run it on a wav file, the output is always different); and
unfortunately this behavior cannot be configured in vanilla soxr.

[This is specially troublesome in ffmpeg](https://ffmpeg-user.ffmpeg.narkive.com/N2TUbPVd/audio-resampler-sox-returns-different-result).

**This project fixes that.**

I've [reported the problem](https://github.com/chirlu/soxr/issues/8) with a
proposed solution. However:

1. It needs to be accepted & merged.
2. soxr needs then to publish a new version.
3. ffmpeg project would then have to pick the new API to fix this behavior.
    - Ideally the seed should be provided through the command line.
4. This could take years (if ever).
5. I don't have that time.

# What does this fix do?

This fork simply hardcodes the seed value; so that it always produces
the same output.

## Is deterministic output better?

No. Whether you need this patch has little to do about quality or speed and
more about guaranteeing each run always produces binary identical files.

If you don't need such guarantee (e.g. realtime communication), then you
don't need this fork.

# How to use this fix?

You need to compile this fork, and then replace the DLLs / so which ffmpeg
is linking against.

I've only tested this on Ubuntu, but it should work on any OS.

## Compiling the project on Linux

```bash
mkdir -p build/Release
cd build/Release
cmake ../../ -DCMAKE_BUILD_TYPE=Release -GNinja
ninja
```

If successfully the following files should be under `soxr/build/Release/src`:

 - libsoxr.so
 - libsoxr.so.0
 - libsoxr.so.0.1.2
 - libsoxr-lsr.so
 - libsoxr-lsr.so.0
 - libsoxr-lsr.so.0.1.9
 
## Patching ffmpeg

On Linux you can use `LD_LIBRARY_PATH` to inject this version of libsoxr
instead of using the system one:

```bash
export LD_LIBRARY_PATH=/path/to/soxr/build/Release/src
ffmpeg -i input.wav -af aresample=48000:resampler=soxr:precision=28 -c:a pcm_s16le output.wav
```

On Windows, copy pasting the DLLs on top of ffmpeg's should work.

# Original Readme

SoX Resampler Library       Copyright (c) 2007-18 robs@users.sourceforge.net

The SoX Resampler library `libsoxr` performs one-dimensional sample-rate
conversion -- it may be used, for example, to resample PCM-encoded audio.
For higher-dimensional resampling, such as for visual-image processing, you
should look elsewhere.

It aims to give fast¹ and very high quality² results for any constant
(rational or irrational) resampling ratio.  Phase-response, preserved
bandwidth, aliasing, and rejection level parameters are all configurable;
alternatively, simple `preset` configurations may be selected.  A
variable-rate resampling mode of operation is also included.

The resampler is currently available either as part of `libsox` (the audio
file-format and effect library), or stand-alone as `libsoxr` (this package).
The interfaces to libsox and libsoxr are slightly different, with that of
libsoxr designed specifically for resampling.  An application requiring
support for other effects, or for reading-from or writing-to audio files or
devices, should use libsox (or other libraries such as libsndfile or
libavformat).

Libsoxr provides a simple API that allows interfacing using the most
commonly-used sample formats and buffering schemes: sample-formats may be
either floating-point or integer, and multiple channels either interleaved
or split in separate buffers.  The API is documented in the header file
`soxr.h`, together with sample code found in the 'examples' directory.

For compatibility with the popular `libsamplerate` library, the header file
`soxr-lsr.h` is provided and may be used as an alternative API.³  Note
however, that libsoxr does not provide a full emulation of libsamplerate
and that using this approach, only a sub-set of libsoxr's features are
available.

The design was inspired by Laurent De Soras' paper `The Quest For The
Perfect Resampler`, http://ldesoras.free.fr/doc/articles/resampler-en.pdf;
in essence, it combines Julius O. Smith's `Bandlimited Interpolation`
technique (https://ccrma.stanford.edu/~jos/resample/resample.pdf) with FFT-
based over-sampling.

Note that for real-time resampling, libsoxr may have a higher latency
than non-FFT based resamplers.  For example, when using the `High Quality'
configuration to resample between 44100Hz and 48000Hz, the latency is
around 1000 output samples, i.e. roughly 20ms (though passband and FFT-
size configuration parameters may be used to reduce this figure).

For build and installation instructions, see the file `INSTALL`; for
copyright and licensing information, see the file `LICENCE`.

For support and new versions, see https://soxr.sourceforge.net
________
¹ For example, multi-channel resampling can utilise multiple CPU-cores.<br/>
² Bit-perfect within practical occupied-bandwidth limits.<br/>
³ For details of that API, see http://www.mega-nerd.com/SRC/api.html.<br/>
