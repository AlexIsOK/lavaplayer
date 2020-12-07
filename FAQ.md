# FAQ and common issues

## Errors

**ALWAYS** include stack traces and the track that was played (in case of an URL) when reporting an error.

### `NoSuchMethodError` and `NoClassDefFoundError`

If Maven is used for building, then this is likely a problem with the way Maven handles version conflicts. When there are two things which require different versions of a dependency, Maven uses the braindead strategy of choosing the one which is the least transitive. This way it will likely choose an older version than the one required by LavaPlayer, which will cause runtime errors.

Unfortunately it is [not possible](https://stackoverflow.com/questions/34201120/maven-set-dependency-mediation-strategy-to-newest-rather-than-nearest) to currently change the version conflict resolution strategy in Maven, so the only solution is to [check the dependency tree](https://maven.apache.org/plugins/maven-dependency-plugin/examples/resolving-conflicts-using-the-dependency-tree.html) and manually declare a dependency on the highest version of the affected package that can be seen in the tree.

An alternative is to use Gradle, which has a sane default (and if necessary, highly configurable) version conflict resolution strategy.

### Error `Something went wrong when...`

This is just a wrapper exception around all other exceptions, check the full stack trace.

### `UnsatisfiedLinkError` <sup><sup><sub><a name="8d9389a5" href="#8d9389a5">8d9389a5</a></sub></sup></sup>

LavaPlayer uses native libraries for some audio encoding and decoding operations. The only platforms that it is compiled for out of the box are:

* Windows x86-64
* Linux x86-64, only compatible with libc, not with musl (Alpine uses musl)
* OS X x86-64

Only file a bug if you are sure you are running the library on a supported platform, but are still getting the exception. Keep in mind that you have to make sure that you are also running a 64-bit JDK. It is a common mistake to run a 32-bit JDK on 64-bit Windows.

For other platforms, there are third-party sources for the native libraries compiled for other platforms. Using those usually involves including an additional dependency in your project.

Third-party provided libraries for support for additional platforms:

* Bundle from various sources
  * Platforms: `linux-aarch32`, `linux-aarch64`, `linux-arm`, `linux-armhf`, `linux-x86`
  * Repository: `https://dl.bintray.com/sedmelluq/com.sedmelluq`
  * Dependency: `com.sedmelluq:lavaplayer-natives-extra:1.3.13`

### YouTube response code 429, or `JsonParseException` <sup><sup><sub><a name="4f89cc01" href="#4f89cc01">4f89cc01</a></sub></sup></sup>

For `JsonParseException`, only applicable if the message starts with `Unexpected character ('<' (code 60))`.

Your IP is being rate limited or blocked by YouTube. The most reliable way to get around this is to get an IPv6 block for your bot and then make use of the `youtube-rotator` plugin. For further details, you could ask in JDA Discord server (also check pins), as there are people around there who are using such a setup.   

## Issues

### Everything triggers `noMatches`.

Did you register source managers? For playing remote tracks (URLs including YouTube):

```
AudioSourceManagers.registerRemoteSources(playerManager);
```

For playing local tracks (files on disk):

```
AudioSourceManagers.registerLocalSource(playerManager);
```

### Sources are registered, but all YouTube tracks trigger `noMatches`.

Make sure you do not convert the input you pass to `AudioPlayerManager#loadItem` to lowercase as YouTube URLs are case-sensitive.

### Playback stutters <sup><sup><sub><a name="45aba0a6" href="#45aba0a6">45aba0a6</a></sub></sup></sup>

In the class that calls `AudioPlayer#provide`, record the number of times it returns a `null`. If the count constantly increases over time (excluding around when a track starts when it is fine to happen), it might be LavaPlayer lagging behind (possibly the CPU is under too much load).

Otherwise, it is an issue with either packet sending or network. Packet sending issues might be caused by garbage collection pauses in the JVM - for JDA this is mitigated by [JDA-NAS](https://github.com/sedmelluq/jda-nas). For other Discord libraries, you should compare with a test bot with JDA+JDA-NAS on the same machine to verify it is not an issue with the library.