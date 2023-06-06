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

checkersd query checkers show-system-info --output json

checkersd query checkers list-stored-game

checkersd tx checkers --help

ignite scaffold message createGame black red \
    --module checkers \
    --response gameIndex

ignite chain serve --reset-once

ignite chain serve

checkersd tx checkers --help
checkersd tx checkers create-game --help

# need to be applied for every time we restart the server.
export alice=$(checkersd keys show alice -a)
export bob=$(checkersd keys show bob -a)

checkersd tx checkers create-game $alice $bob --from $alice --dry-run

checkersd tx checkers create-game $alice $bob --from $alice --gas auto

echo YWN0aW9u | base64 -d
echo Y3JlYXRlX2dhbWU= | base64 -d
echo c2lnbmF0dXJl | base64 -d

checkersd keys add alice --keyring-backend test
checkersd keys add bob --keyring-backend test

# Troubleshooting - key not found
export alice=$(checkersd keys show alice -a --keyring-backend test)
export bob=$(checkersd keys show bob -a --keyring-backend test)
checkersd tx \
    checkers create-game \
    $alice $bob \
    --from $alice \
    --gas auto \
    --keyring-backend test

checkersd query checkers show-system-info
checkersd query checkers list-stored-game

# Test transaction to create game
ignite chain serve --reset-once
export alice=$(checkersd keys show alice -a)
export bob=$(checkersd keys show bob -a)
checkersd tx checkers create-game $alice $bob --from $alice --gas auto
checkersd query checkers show-system-info
checkersd query checkers list-stored-game
checkersd query checkers show-stored-game 1
checkersd query checkers show-stored-game 1 --output json | jq ".storedGame.board" | sed 's/"//g' | sed 's/|/\'$'\n/g'
```
