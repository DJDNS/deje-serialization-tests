deje-serialization-tests
========================

A bash-based language-agnostic way to test conformance to DEJE's deterministic JSON serialization rules.

The source code is almost simple enough to act as all the documentation that the project needs, and you can also look at the way it's used in [other](https://github.com/DJDNS/go-deje) [projects](https://github.com/DJDNS/js-deje) for reference. But it's probably worth some explanation anyways.

The deje-serialization-tests/run script is a test harness. You provide it with a command to run, and it repeatedly runs that command to generate test output. For example:

    ./run echo # Use 'echo' program as test hook.

This will repeatedly call 'echo', piping in some JSON data, and piping the output to disk in the appropriate location. You can also run a specific test:

    ./run 'echo' abcd model pretty4

The entire command must be provided as one ARGV variable, using quoting if necessary, because any later arguments will be interpreted as naming a specific test to run.

### Calling the hook

Whenever you use this test suite in another project, you'll do it by implementing a hook, which ./run will call repeatedly with the environment set up accordingly. All your hook needs to do is understand a few arguments, read stdin, and print to stdout.

    $COMMAND $object_type $format < input.json > output.json

This is a bit simplified, but the gist of what happens. Your command (which may contain arguments) is run with 2 additional arguments, and standard fds set up. After running each test, the output is compared against the expected output.

#### `$object_type`

Generally, all JSON {} objects should be serialized with sorted keys, in order to serialize deterministically. However, some object types have a specific order in which their keys should be presented.

```
event:
{
    "parent": "...",
    "handler": "...",
    "args": {...}
}

document:
{
    "topic": "...",
    "events": {...},
    "quorums": {...},
    "timestamps": [...]
}
```

This is why the hook needs context about what kind of object it should be constructing and serializing.

#### `$format`

There are currently 3 formats that a hook may be expected to serialize to:

 * _pretty4_: Whitespace-y JSON. 4 space indent, 1 space between key and value in {}. Good for debugging.
 * _compact_: JSON with whitespace trimmed out. This is what gets hashed.
 * _hash_: To verify the SHA1 implementation.

For both the JSON formats, key ordering rules must be observed.

#### input.json

Your hook is fed JSON data via STDIN. This will always be valid JSON, we are not trying to test anyone's error handling.

#### output.json

Finally, a file is populated from the hook's STDOUT.

**THIS SHOULD NEVER OUTPUT A NEWLINE AT THE END OF THE FILE.** It's cleaner this way, since the compact content should not include a newline when hashed, and we might as well be consistent.
