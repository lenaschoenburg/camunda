# Maven Build Cache Extension

This project uses the [Apache Maven Build Cache Extension](https://maven.apache.org/extensions/maven-build-cache-extension/) to speed up builds by caching outputs from previous builds.

## Overview

The Maven Build Cache Extension is configured to:
- Cache build outputs locally in `~/.m2/build-cache`
- Use SHA-256 hash algorithm for cache key computation
- Support multi-module project builds
- Exclude IDE and OS-specific files from cache calculations
- Keep up to 100 cached builds

## How It Works

The extension works by:
1. Computing a hash of the inputs (source code, dependencies, plugin configurations)
2. Checking if a cached build exists for that hash
3. If found, restoring the cached outputs instead of rebuilding
4. If not found, performing a normal build and caching the outputs

When a module is restored from cache, you'll see messages like:

```
[INFO] Found cached build, restoring io.camunda:module-name from cache by checksum abc123...
[INFO] Skipping plugin execution (cached): compiler:compile
```

## Configuration Files

### `.mvn/extensions.xml`

Declares the Maven Build Cache Extension (version 1.2.0) along with other Maven extensions.

### `.mvn/maven-build-cache-config.xml`

Contains the cache configuration including:
- Hash algorithm (SHA-256)
- Cache location and size limits
- Multi-module discovery settings
- Input exclusions (IDE files, target directories, etc.)
- Plugin-specific configurations

## Build Commands

The cache extension works transparently with all Maven commands:

```bash
# Quick build (uses cache)
./mvnw install -Dquickly -T1C

# Full build (uses cache)
./mvnw clean install

# Build with specific modules
./mvnw install -pl :module-name -am

# Parallel builds work with cache
./mvnw install -T1C
```

## Cache Management

### Viewing Cache Information

The cache is stored in `~/.m2/build-cache/` by default. You can check its size:

```bash
du -sh ~/.m2/build-cache/
```

### Clearing the Cache

To clear the local cache:

```bash
rm -rf ~/.m2/build-cache/
```

The cache will automatically maintain up to 100 cached builds and remove older entries.

### Disabling the Cache

To temporarily disable the cache for a single build:

```bash
./mvnw install -Dmaven.build.cache.enabled=false
```

## Understanding Cache Behavior

### Cache Hits

A cache hit occurs when:
- Source code hasn't changed
- Dependencies haven't changed
- Plugin configurations haven't changed
- Build tool versions haven't changed

### Cache Misses

A cache miss occurs when:
- Any source file is modified
- Dependencies are updated
- Plugin configurations change
- First time building a module

### Expected Behavior

- **First build**: No cache hits, all modules build normally
- **Second build (no changes)**: Most modules restored from cache
- **After changing a module**: Only that module and its dependents rebuild
- **After `mvn clean`**: Cache is still preserved and will be used

## Troubleshooting

### "Cannot initialize cache" Error

If you see this error, check that:
- The `.mvn/maven-build-cache-config.xml` file is valid XML
- The schema version matches the extension version (1.2.0)
- File permissions allow writing to `~/.m2/build-cache/`

### Unexpected Rebuilds

If modules are rebuilding when you expect cache hits:
- Check if you modified any files (including comments or whitespace)
- Verify that your IDE isn't modifying files automatically
- Check if environment variables or system properties changed

### Build Failures with Cache

If a build fails with the cache enabled but works without it:
1. Clear the cache: `rm -rf ~/.m2/build-cache/`
2. Report the issue with details about the failure

## Performance Impact

Expected improvements with a warm cache:
- **Clean builds with no changes**: 60-80% faster (only dependency resolution and cache restoration)
- **Builds with few changes**: 40-60% faster (only changed modules rebuild)
- **Parallel builds**: Enhanced by cache (unchanged modules skip immediately)

The actual improvement depends on:
- Number of modules in the build
- Number of modules that changed
- Complexity of the build
- I/O performance of the cache storage

## Remote Cache (CI/CD)

The configuration currently has remote cache disabled. To enable remote caching for CI/CD:

1. Set up a remote cache server (HTTP/WebDAV, Nexus, or Artifactory)
2. Update `.mvn/maven-build-cache-config.xml`:

   ```xml
   <remote enabled="true">
     <url>https://your-cache-server.com/maven-cache</url>
   </remote>
   ```
3. Configure authentication if needed

Remote cache allows multiple developers and CI builds to share cached artifacts.

## Best Practices

1. **Don't commit cache artifacts**: The cache is local and shouldn't be in version control
2. **Keep cache config in sync**: Changes to `.mvn/maven-build-cache-config.xml` affect all developers
3. **Monitor cache size**: The cache is limited to 100 builds by default
4. **Use with parallel builds**: Combine `-T1C` with cache for maximum speed
5. **Clear cache on Maven upgrades**: Major Maven or plugin upgrades may benefit from a fresh cache

## References

- [Apache Maven Build Cache Extension Documentation](https://maven.apache.org/extensions/maven-build-cache-extension/)
- [Getting Started Guide](https://maven.apache.org/extensions/maven-build-cache-extension/getting-started.html)
- [Configuration Reference](https://maven.apache.org/extensions/maven-build-cache-extension/build-cache-config.html)

