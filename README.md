# ExifToolHelper
Java wrapper for [ExifTool](https://exiftool.org) used in my other repos. Current version: v0.1

ExifTool is started once with `-stay_open` and then gets all commands through that one process - much faster than one ExifTool call per file. No JavaFX, so it can be used from command line tools as well. The only dependency is Jackson for the JSON output of ExifTool.

## Usage

```java
try (ExifTool exifTool = new ExifTool(Path.of("C:/Tools/exiftool.exe"))) {
    System.out.println(exifTool.version());

    // selected tags of several files: "-json -n -G1", i.e. numeric values and keys like "Composite:GPSLatitude"
    List<Map<String, Object>> tags = exifTool.readJson(List.of(photo), List.of("Composite:GPSLatitude", "XMP-photoshop:City"));

    // all tags of one file, human readable - e.g. for an "image info" view
    Map<String, String> allTags = exifTool.readAllTags(photo);

    // embedded images, e.g. the preview of a RAW file
    boolean found = exifTool.extractBinary(photo, "PreviewImage", Path.of("preview.jpg"));

    // anything else
    ExecResult result = exifTool.execute(List.of("-overwrite_original", "-XMP-photoshop:City=Meran", photo.toString()));
}
```

`ExifTool` is thread safe: commands from several threads are executed one after the other. If the process crashes or doesn't answer within the timeout (default 60 seconds), it is restarted and the command is retried once.

## Notes

* Note on building: "gradlew build" compiles, tests and installs the jar into the local maven repo. Found there by the other repos during their build as `tf.ExifToolHelper:ExifToolHelper:<version>`. Requires JDK 25; Gradle comes via the wrapper.
* Note on tests: tests that need ExifTool look for it in the environment variable EXIFTOOL_PATH and then on the PATH - without it they are skipped.
