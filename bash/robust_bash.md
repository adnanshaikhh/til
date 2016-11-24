## Quit if variable is unset while substituting
```bash
set -u
set -o nounset
```

## Surround filenames with quotes
```bash
if [ "$filename" = "foo" ];
```