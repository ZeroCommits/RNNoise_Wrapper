# RNNoise Wrapper

This is a simple Python wrapper for [RNNoise noise
reduction](https://github.com/xiph/rnnoise) . Only Python 3 is
supported. The code is based on [an issue from
snakers4](https://github.com/xiph/rnnoise/issues/69) from the RNNoise
repository, for which special thanks to him.

[RNNoise](https://jmvalin.ca/demo/rnnoise/) is a recurrent neural
network with GRU cells designed for real-time audio denoising (even
works on Raspberry Pi). The standard model is trained on 6.4GB of noisy
audio recordings and is completely ready for use.

RNNoise is written in C and has methods for denoising a single 10
millisecond frame. The frame must have a sampling frequency of 48000Hz,
mono, 16 bit.

**RNNoise_Wrapper** makes working with RNNoise easy:

-   eliminates the need to extract frames from the audio recording
    yourself
-   removes restrictions on the parameters of the processed wav audio
    recording
-   hides all the nuances of working with a C library
-   eliminates the need to manually compile RNNoise (Linux only)
-   adds 2 new binaries with higher quality models that come with the
    package (Linux only)

**RNNoise_Wrapper contains 2 new higher quality models** (trained
weights and compiled RNNoise binaries for Linux). A dataset from
[Microsoft DNS Challenge](https://github.com/microsoft/DNS-Challenge)
was used for training .

1.  **librnnoise_5h_ru_500k** - trained on 5 hours of Russian speech
    (mixed with emotional speech and singing in English), obtained by a
    script from a repository with a dataset. Trained weights are in
    [`train_logs/weights_5h_ru_500k.hdf5 `](https://github.com/Desklop/RNNoise_Wrapper/tree/master/train_logs/weights_5h_ru_500k.hdf5),
    compiled by RNNoise in
    [`rnnoise_wrapper/libs/librnnoise_5h_ru_500k.so.0.4.1 `](https://github.com/Desklop/RNNoise_Wrapper/tree/master/rnnoise_wrapper/libs/librnnoise_5h_ru_500k.so.0.4.1)(Linux
    only)

2.  **librnnoise_5h_b_500k** - trained in 5 hours of mixed speech in
    English, Russian, German, French, Italian, Spanish and Mandarin
    Chinese (mixed with emotional speech and singing in English). The
    dataset for each language was previously trimmed to the smallest of
    them (the least amount of data is for the Russian language, about 47
    hours). The final training sample was obtained by a script from the
    repository with the dataset. Trained weights are in
    [`train_logs/weights_5h_b_500k.hdf5 `](https://github.com/Desklop/RNNoise_Wrapper/tree/master/train_logs/weights_5h_b_500k.hdf5),
    compiled by RNNoise in
    [`rnnoise_wrapper/libs/librnnoise_5h_b_500k.so.0.4.1 `](https://github.com/Desklop/RNNoise_Wrapper/tree/master/rnnoise_wrapper/libs/librnnoise_5h_b_500k.so.0.4.1)(Linux
    only)

3.  **librnnoise_default** - standard model from the authors [of
    RNNoise](https://jmvalin.ca/demo/rnnoise/)

Models `librnnoise_5h_ru_500k `and `librnnoise_5h_b_500k `have **almost
the same** noise reduction quality. `librnnoise_5h_ru_500k is `most
**suitable for working with Russian speech** , and
`librnnoise_5h_b_500k `is most **suitable for mixed speech** or speech
in a non-Russian language, it is more universal.

Comparative examples of the new models working with the standard ones
are available in
[`test_audio/comparative_tests `](https://github.com/Desklop/RNNoise_Wrapper/tree/master/test_audio/comparative_tests).

This wrapper on the Intel i7-10510U CPU **works 28-30 times faster than
real time** when denoising an entire audio recording, and **18-20 times
faster than real time** when working in streaming mode (i.e. processing
audio fragments 20 ms long). In this case, only 1 core was used, the
load on which was about 80-100%.

## Installation

This wrapper has the following dependencies:
[pydub](https://github.com/jiaaro/pydub) and
[numpy](https://github.com/numpy/numpy) .

Installation using pip:

    pip install git+https://github.com/Desklop/RNNoise_Wrapper

**ATTENTION!** Before using the wrapper, RNNoise must be compiled. If
you are using **Linux or Mac** , you can use **the pre-compiled
RNNoise** (on Ubuntu 19.10 64 bit OS) that **comes with the package**
(it also works on Google Colaboratory). If the standard binary doesn\'t
work for you, try compiling RNNoise manually. To do this you need to
first prepare your OS (assuming `gcc `is already installed):

    sudo apt-get install autoconf libtool

And execute:

    git clone https://github.com/Desklop/RNNoise_Wrapper 
    cd RNNoise_Wrapper 
    ./compile_rnnoise.sh

After this, the `librnnoise_default.so.0.4.1 `file will appear in the
`rnnoise_wrapper/libs folder `. The path to this binary file must be
passed when creating an object of the RNNoise class from this wrapper
(see below for more details).

If you are using **Windows** , then you need to **manually compile
RNNoise** . The above instructions will not work, **use** these
**links** : [one](https://github.com/xiph/rnnoise/issues/34) ,
[two](https://github.com/jagger2048/rnnoise-windows) . After
compilation, the path to the binary file must be passed when creating an
object of the RNNoise class from this wrapper (see below for more
details).

## Usage

### **1. In Python code**

**Reduce noise in audio recordings** `test.wav `and saving the result as
`test_denoised.wav `:

    from rnnoise_wrapper import RNNoise 

    denoiser = RNNoise() 

    audio = denoiser.read_wav( 'test.wav' ) 
    denoised_audio = denoiser. filter (audio) 
    denoiser.write_wav( 'test_denoised.wav' , denoised_audio)

**Noise reduction in streaming audio** (buffer size is 20 milliseconds,
i.e. 2 frames) (the example uses stream simulation by processing the
`test.wav audio recording `in parts and saving the result as
`test_denoised_stream.wav `):

    audio = denoiser.read_wav('test.wav')

    denoised_audio = b''
    buffer_size_ms = 20

    for i in range(buffer_size_ms, len(audio), buffer_size_ms):
        denoised_audio += denoiser.filter(audio[i-buffer_size_ms:i].raw_data, sample_rate=audio.frame_rate)
    if len(audio) % buffer_size_ms != 0:
        denoised_audio += denoiser.filter(audio[len(audio)-(len(audio)%buffer_size_ms):].raw_data, sample_rate=audio.frame_rate)

    denoiser.write_wav('test_denoised_stream.wav', denoised_audio, sample_rate=audio.frame_rate)

**More examples of working with a wrapper** can be found in
[`rnnoise_wrapper_functional_tests.py `](https://github.com/Desklop/RNNoise_Wrapper/blob/master/rnnoise_wrapper_functional_tests.py)and
[`rnnoise_wrapper_comparative_test.py `](https://github.com/Desklop/RNNoise_Wrapper/blob/master/rnnoise_wrapper_comparative_test.py).

[RNNoise](https://github.com/Desklop/RNNoise_Wrapper/blob/master/rnnoise_wrapper/rnnoise_wrapper.py#L29)
class contains the following methods:

-   [`read_wav() `](https://github.com/Desklop/RNNoise_Wrapper/blob/master/rnnoise_wrapper/rnnoise_wrapper.py#L256):
    takes the name of a .wav audio recording, converts it to a supported
    format (16-bit, mono) and returns a
    `pydub.AudioSegment object `containing the audio recording
-   [`write_wav() : takes the .wav name of an audio recording, a `](https://github.com/Desklop/RNNoise_Wrapper/blob/master/rnnoise_wrapper/rnnoise_wrapper.py#L277)`pydub.AudioSegment `object
    (or a byte string of audio data without wav headers) and saves the
    audio recording under the passed name
-   [`filter() `](https://github.com/Desklop/RNNoise_Wrapper/blob/master/rnnoise_wrapper/rnnoise_wrapper.py#L150):
    takes a `pydub.AudioSegment object `(or a byte string of audio data
    without wav headers), resamples it to 48000 Hz, **splits the audio
    into frames** (10 milliseconds long), **denoises them, and returns
    a** `pydub.AudioSegment `object (or byte string without wav headers)
    preserving the original sampling rate
-   [`filter_frame() `](https://github.com/Desklop/RNNoise_Wrapper/blob/master/rnnoise_wrapper/rnnoise_wrapper.py#L128):
    clear only one frame (10 ms long, 16 bit, mono, 48000 Hz) from noise
    (accessing the RNNoise library binary directly)

Detailed information about the supported arguments and operation of each
method can be found in the comments in the source code of those methods.

**The default model is** `librnnoise_5h_b_500k `. When creating an
object of the `RNNoise class `from a wrapper, using the
`f_name_lib argument `, you can specify a different model (RNNoise
binary):

-   `librnnoise_5h_ru_500k `or `librnnoise_default `to use one of the
    bundled models
-   full/partial name/path to the compiled RNNoise binary file

```{=html}
<!-- -->
```
    denoiser_def = RNNoise(f_name_lib = 'librnnoise_5h_ru_500k' ) 
    denoiser_new = RNNoise(f_name_lib = 'path/to/librnnoise.so.0.4.1' )

**Features of the main** `filter() method `**:**

-   For the highest quality work, you need an audio recording of at
    least 1 second in length, which contains both voice and noise (and
    noise should ideally be before and after the voice). Otherwise, the
    quality of noise reduction will be worse
-   in case parts of one audio recording are transmitted (streaming
    audio noise reduction), then their length must be at least `10 `ms
    and a multiple of `10 `(since the RNNoise library only supports
    frames with a length of `10 `ms). This option does not affect the
    quality of noise reduction.
-   if the last frame of the transmitted audio recording is less than
    `10 `ms (or a part of the audio less than `10 `ms long is
    transmitted), then it is padded with zeros to the required size.
    Because of this, there may be a slight increase in the length of the
    final audio recording after noise reduction
-   The RNNoise library additionally returns for each frame the
    probability of having a voice in this frame (as a number from `0 `to
    `1 `) and using the `voice_prob_threshold argument `you can filter
    frames by this value. If the probability is lower than
    `voice_prob_threshold `, then the frame will be removed from the
    audio recording

### **2. As a command line tool**

    python3 -m rnnoise_wrapper.cli -i input.wav -o output.wav

or

    rnnoise_wrapper -i input.wav -o output.wav

Where:

-   `input.wav `- name of the original .wav audio recording
-   `output.wav `- the name of the .wav audio file into which the audio
    recording will be saved after noise reduction

## Education

Instructions for training RNNoise using your own data can be found at
[`TRAINING.md `](https://github.com/Desklop/RNNoise_Wrapper/tree/master/TRAINING.md).

If you have any questions or want to collaborate, you can write to me by
email: vladsklim@gmail.com or on
[LinkedIn](https://www.linkedin.com/in/vladklim/) .
