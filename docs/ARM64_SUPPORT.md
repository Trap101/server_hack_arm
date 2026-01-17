# ARM64 Support

This project now supports building and running on both `linux/amd64` (x86-64) and `linux/arm64` (ARM 64-bit) architectures.

## Building for ARM64

### Local Development

Build for a specific architecture:

```bash
# Build for ARM64
make build BUILD_PLATFORM=linux/arm64

# Build for AMD64 (default)
make build BUILD_PLATFORM=linux/amd64
```

### Multi-Platform Builds

Build for both architectures and push to registry:

```bash
# Requires docker buildx and Docker registry credentials
make build-multiplatform IMAGE_VERSION=1.4.2
```

## Architecture Support Matrix

| Component | AMD64 (x86-64) | ARM64 (aarch64) | Notes |
|-----------|---|---|---|
| .NET SDK/Runtime | ✅ | ✅ | Full support |
| Main Server Image | ✅ | ✅ | Uses jlesage/baseimage-gui |
| Steam-Service | ✅ | ✅ | Requires libssl-dev for SteamKit2 |
| XNB-Unpacker | ✅ | ✅ | Uses Debian base image |
| CI/CD Builds | ✅ | ✅ | Automated via GitHub Actions |

## Technical Details

### Main Dockerfile Changes

- Added `ARG TARGETARCH` and `ARG TARGETOS` for cross-architecture builds
- DLL-patcher now builds with `--runtime linux-$TARGETARCH`
- tmux binary download detects architecture and downloads correct binary

### Steam-Service Changes

- Added `libssl-dev` installation for native OpenSSL support (required for SteamKit2)
- Added `RuntimeIdentifiers` to project file for explicit architecture support

### Makefile Changes

- Added `BUILD_PLATFORM` variable (default: linux/amd64)
- Added `build-multiplatform` target for multi-architecture builds
- Existing `build` target now respects `BUILD_PLATFORM` variable

### GitHub Actions Changes

- All Docker build workflows now include `platforms: linux/amd64,linux/arm64`
- Applies to: release, preview, steam-service release, steam-service preview

## Deployment

Docker Hub automatically serves the correct architecture when pulling images:

```bash
# Automatically pulls amd64 on x86-64 systems, arm64 on ARM systems
docker pull sdvd/server:latest
docker pull sdvd/steam-service:latest
```

## Testing ARM64 Locally

If you have an ARM-based system:

```bash
# Run the server
docker compose up -d

# The correct architecture image will be pulled automatically
```

If you're on x86-64 and want to test ARM64 (slow, requires QEMU):

```bash
# Build for ARM64 (requires docker buildx)
docker buildx build --platform linux/arm64 -t sdvd/server:test-arm64 .
```

## Troubleshooting

### Build fails with "architecture not supported"

Check that your Docker setup supports the target architecture:

```bash
docker buildx ls
# Should show available platforms
```

### Steam-service fails with "libssl not found"

Ensure the Docker image includes `libssl-dev`:

```bash
docker inspect sdvd/steam-service:latest | grep -i ssl
```

### Multi-platform builds fail

Ensure you have docker buildx configured:

```bash
docker buildx create --use
```

## Related

- [Dockerfile](../docker/Dockerfile) - Multi-stage build with ARM64 support
- [Makefile](../Makefile) - Build automation with multi-platform support
- [GitHub Actions Workflows](.github/workflows/) - CI/CD with ARM64 builds
