# Info

## Ignite CLI with scaffold

```sh
ignite scaffold chain github.com/alice/checkers

cd checkers
ignite chain serve

mkdir x/checkers/rules
curl https://raw.githubusercontent.com/batkinson/checkers-go/a09daeb1548dd4cc0145d87c8da3ed2ea33a62e3/checkers/checkers.go | sed 's/package checkers/package rules/' > x/checkers/rules/checkers.go

ignite scaffold single systemInfo nextId:uint \
    --module checkers \
    --no-message

ignite scaffold map storedGame board turn black red \
    --index index \
    --module checkers \
    --no-message

ignite generate proto-go

go test github.com/alice/checkers/x/checkers/keeper

ignite chain serve --reset-once

checkersd query checkers --help

checkersd query checkers show-system-info
```
