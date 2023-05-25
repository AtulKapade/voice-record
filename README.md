# voice-record
voice record for 5 s 
#include <stdio.h>
#include <stdlib.h>
#include <portaudio.h>

#define SAMPLE_RATE 44100
#define FRAMES_PER_BUFFER 512

// Callback function for audio input
int audioInputCallback(const void* inputBuffer, void* outputBuffer,
                       unsigned long framesPerBuffer,
                       const PaStreamCallbackTimeInfo* timeInfo,
                       PaStreamCallbackFlags statusFlags,
                       void* userData)
{
    // Cast the input buffer to a float pointer
    const float* samples = (const float*)inputBuffer;

    // Process or store the audio samples here
    // In this example, we'll print the RMS value of the samples
    float rms = 0.0;
    for (int i = 0; i < framesPerBuffer; ++i)
    {
        rms += samples[i] * samples[i];
    }
    rms = sqrtf(rms / framesPerBuffer);

    printf("RMS: %f\n", rms);

    return paContinue;
}

int main()
{
    PaStream* stream;
    PaError err;

    // Initialize PortAudio
    err = Pa_Initialize();
    if (err != paNoError)
    {
        printf("PortAudio initialization failed: %s\n", Pa_GetErrorText(err));
        return -1;
    }

    // Open a stream for audio input
    err = Pa_OpenDefaultStream(&stream, 1, 0, paFloat32, SAMPLE_RATE,
                               FRAMES_PER_BUFFER, audioInputCallback, NULL);
    if (err != paNoError)
    {
        printf("Failed to open audio stream: %s\n", Pa_GetErrorText(err));
        Pa_Terminate();
        return -1;
    }

    // Start the stream
    err = Pa_StartStream(stream);
    if (err != paNoError)
    {
        printf("Failed to start audio stream: %s\n", Pa_GetErrorText(err));
        Pa_CloseStream(stream);
        Pa_Terminate();
        return -1;
    }

    printf("Recording audio...\n");
    printf("Press Enter to stop recording.\n");
    getchar();

    // Stop and close the stream
    err = Pa_StopStream(stream);
    if (err != paNoError)
    {
        printf("Failed to stop audio stream: %s\n", Pa_GetErrorText(err));
    }

    err = Pa_CloseStream(stream);
    if (err != paNoError)
    {
        printf("Failed to close audio stream: %s\n", Pa_GetErrorText(err));
    }

    // Terminate PortAudio
    Pa_Terminate();

    return 0;
}
