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

# Add a way to make a move
ignite scaffold message playMove gameIndex fromX:uint fromY:uint toX:uint toY:uint \
    --module checkers \
    --response capturedX:int,capturedY:int,winner

# Test transaction to play move
ignite chain serve --reset-once
export alice=$(checkersd keys show alice -a)
export bob=$(checkersd keys show bob -a)
checkersd tx checkers create-game $alice $bob --from $alice --gas auto
checkersd tx checkers play-move --help
checkersd tx checkers play-move 1 0 5 1 4 --from $bob
checkersd query tx A8ACE948942827B49C102B791F008D6BE21ED9F712F1B805758CA69426EAAFF6
checkersd tx checkers play-move 1 7 2 8 3 --from $alice
checkersd tx checkers play-move 1 1 0 0 1 --from $alice
checkersd tx checkers play-move 1 1 2 2 3 --from $alice
checkersd query checkers show-stored-game 1 --output json | jq ".storedGame.board" | sed 's/"//g' | sed 's/|/\'$'\n/g'

# Test events emit
ignite chain serve --reset-once
export alice=$(checkersd keys show alice -a)
export bob=$(checkersd keys show bob -a)
checkersd tx checkers create-game $alice $bob --from $alice --gas auto
checkersd tx checkers play-move 1 1 2 2 3 --from $alice
checkersd tx checkers play-move 1 0 5 1 4 --from $bob
checkersd query tx 1C9A05A14906A4F94798B8A3A84E35F74DE4F474C3FC8DE9285484C227D911D0 --output json | jq ".raw_log | fromjson"
checkersd query checkers show-stored-game 1 --output json | jq ".storedGame.board" | sed 's/"//g' | sed 's/|/\'$'\n/g'
checkersd tx checkers play-move 1 2 3 0 5 --from $alice

# Generate proto changes
ignite generate proto-go

# Interact with cli to test winner
ignite chain serve --reset-once
export alice=$(checkersd keys show alice -a)
export bob=$(checkersd keys show bob -a)
checkersd tx checkers create-game $alice $bob --from $alice
checkersd query checkers show-stored-game 1
checkersd tx checkers play-move 1 1 2 2 3 --from $alice
checkersd query checkers show-stored-game 1
```