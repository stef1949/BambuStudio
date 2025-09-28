# macOS Virtual Camera Fix for BambuStudio

## Problem

The virtual camera functionality in BambuStudio was not working properly on macOS, especially on macOS 15 (Sequoia) and later versions, when trying to stream to OBS or other applications.

## Root Cause

The issue was caused by incompatibility with the POSIX System V shared memory implementation used for inter-process communication between BambuStudio and the virtual camera driver. On newer macOS versions, these APIs have stricter security restrictions and different behavior.

## Solution

### 1. Enhanced Shared Memory Support

- **POSIX Shared Memory**: Added support for POSIX shared memory (`shm_open`/`mmap`) as the primary method on macOS
- **System V Fallback**: Maintained System V shared memory (`shmget`/`shmat`) as fallback for older macOS versions
- **File-based Fallback**: Added file-based status checking when both shared memory methods fail

### 2. Better Error Handling

- Added comprehensive error logging with `BOOST_LOG_TRIVIAL(trace)` messages
- Implemented macOS-specific error messages and troubleshooting guidance
- Added interactive troubleshooting dialog with step-by-step instructions

### 3. Process Communication Enhancement

- Modified the `bambu_source` executable launch to include `--use-posix-shm` parameter on macOS
- Enhanced process startup with better error detection and reporting

## Usage Instructions

### For Users

1. **Basic Setup**:
   - Ensure BambuStudio has camera access in System Preferences → Security & Privacy → Privacy → Camera
   - Grant screen recording permissions in System Preferences → Security & Privacy → Privacy → Screen Recording

2. **Enable Virtual Camera**:
   - Connect to your Bambu printer
   - Start live view in BambuStudio
   - Click the virtual camera toggle button in the status panel
   - If prompted, allow BambuStudio to download camera tools

3. **OBS Integration**:
   - In OBS Studio, add a new source → Video Capture Device
   - Select "BambuStudio Virtual Camera" as the device
   - Configure video settings as needed

### Troubleshooting

If virtual camera fails to start, BambuStudio will now display a helpful dialog with:
- Permission requirements
- macOS version-specific guidance
- OBS integration steps
- Advanced troubleshooting for persistent issues

## Technical Details

### Code Changes

1. **MediaPlayCtrl.cpp**:
   - Enhanced `get_stream_url()` function with multi-tier fallback strategy
   - Modified `start_stream_service()` to pass macOS-specific parameters
   - Added comprehensive error handling and logging

2. **MediaPlayCtrl.h**:
   - Added macOS-specific troubleshooting function declaration

### Compatibility

- **macOS 15+**: Uses POSIX shared memory for maximum compatibility
- **macOS 10.15-14**: Uses System V shared memory with POSIX fallback
- **Other platforms**: Unchanged behavior (Windows uses named pipes, Linux uses System V)

### File Structure

The virtual camera system uses these files in the BambuStudio data directory:
- `cameratools/url.txt`: Primary communication channel
- `cameratools/status.txt`: Fallback status file
- `cameratools/bambu_source`: Virtual camera executable
- `cameratools/ffmpeg`: Stream processing executable

## Testing

The fix has been tested with:
- Multiple shared memory implementations
- Error condition handling
- Process communication scenarios
- Permission edge cases

## Future Improvements

1. **Native macOS AVCaptureDevice Integration**: Could replace external processes with native macOS camera APIs
2. **Enhanced Security**: Code signing and notarization for camera tools
3. **Performance Optimization**: Direct memory mapping for lower latency streaming

## Notes for Developers

When making changes to the virtual camera system:
1. Always test on multiple macOS versions (especially latest)
2. Consider security implications of shared memory and process communication
3. Ensure proper cleanup of shared memory resources
4. Add appropriate logging for debugging