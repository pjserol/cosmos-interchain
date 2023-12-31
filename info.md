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
checkersd tx checkers create-game $alice $bob 33 --from $alice
checkersd query checkers show-stored-game 1
checkersd tx checkers play-move 1 1 2 2 3 --from $alice
checkersd query checkers show-stored-game 1
```

```sh
# Exercise 2 - interchain
ignite scaffold single WorldParams name gravity:uint landSupply:uint \
    --module otherworld \
    --no-message
```

```sh
# Add deadline to proto
ignite generate proto-go

# Check project still workinkg
ignite chain build
```

```sh
# Add moveCount to proto
ignite generate proto-go

# Check project still workinkg
ignite chain build
```

```sh
# Test FIFO
checkersd query checkers show-system-info

checkersd tx checkers create-game $alice $bob --from $bob
checkersd query checkers show-system-info

checkersd query checkers show-stored-game 1

checkersd tx checkers create-game $alice $bob --from $bob
checkersd query checkers show-system-info

checkersd query checkers show-stored-game 1
checkersd query checkers show-stored-game 2

checkersd tx checkers play-move 2 1 2 2 3 --from $alice
checkersd query checkers show-system-info

checkersd tx checkers create-game $alice $bob --from $alice
checkersd tx checkers play-move 3 1 2 2 3 --from $alice
checkersd tx checkers play-move 3 0 5 1 4 --from $bob

checkersd query account $alice --output json | jq -r '.sequence'
checkersd query checkers show-stored-game 3 --output json | jq '.storedGame.winner'
checkersd query checkers show-system-info
```

```sh
# Interact with cli to test wager (need one new argument to create the game)
ignite chain serve --reset-once
export alice=$(checkersd keys show alice -a)
export bob=$(checkersd keys show bob -a)
checkersd tx checkers create-game $alice $bob 1000000 --from $alice
checkersd query checkers show-stored-game 1
checkersd tx checkers play-move 1 1 2 2 3 --from $alice
checkersd query checkers show-stored-game 1
checkersd tx checkers play-move 1 0 5 1 4 --from $bob

checkersd query bank balances $alice
checkersd query bank balances $bob

# Check gas, depending on gas price
# checkersd tx checkers create-game $alice $bob 1000000 --from $alice --dry-run
checkersd tx checkers create-game \
    $alice $bob 1000000 \
    --from $alice -y | \
        grep gas_used

```

```sh
# Prepare mocks
go install github.com/golang/mock/mockgen@v1.6.0

mockgen -source=x/checkers/types/expected_keepers.go \
    -package testutil \
    -destination=x/checkers/testutil/expected_keepers_mocks.go

# If use docker, add this line to automate it
ENV PACKAGES curl gcc jq make

# If any changes, use the command to regenerate mocks
make mock-expected-keepers
```

## Query messages

```sh
ignite scaffold query canPlayMove \
    gameIndex player fromX:uint fromY:uint toX:uint toY:uint \
    --module checkers \
    --response possible:bool,reason
```

## Query play move

```sh
checkersd query checkers can-play-move --help
checkersd query checkers can-play-move 2048 r 1 2 2 3
checkersd query checkers can-play-move 1 w 1 2 2 3
checkersd query checkers can-play-move 1 b 1 2 2 3
checkersd query checkers can-play-move 1 b 2 3 3 4
```

## Add token

```sh
# Interact with cli to test wager (need one new argument to create the game)
ignite chain serve --reset-once
export alice=$(checkersd keys show alice -a)
export bob=$(checkersd keys show bob -a)
checkersd tx checkers create-game $alice $bob 1000000 coin --from $alice
checkersd query checkers show-stored-game 1
```

## IBC Denoms - Gaia

```sh
git clone https://github.com/cosmos/gaia.git
cd gaia
git checkout v7.0.0
make install

gaiad version

# If not available, look at registry https://github.com/cosmos/chain-registry/blob/master/cosmoshub/chain.json
gaiad query ibc-transfer denom-trace 14F9BC3E44B8A9C1BE1FB08980FAB87034C9905EF17CF2F5008FC085218811CC --node https://rpc.cosmos.network:443

gaiad query ibc-transfer denom-trace 14F9BC3E44B8A9C1BE1FB08980FAB87034C9905EF17CF2F5008FC085218811CC --node https://rpc-cosmoshub.blockapsis.com:443

gaiad query ibc channel client-state transfer channel-141 --node https://rpc-cosmoshub.blockapsis.com:443
```

## Go Relayer

```sh
mkdir relay-go-test
cd relay-go-test
git clone https://github.com/cosmos/relayer.git

cd relayer
make install

rly -h

rly config init

rly config show

rly config init --memo "My custom memo"

rly chains add cosmoshub osmosis
```

## Generate protobuf

```sh
mkdir -p scripts/protoc
cd scripts/protoc
curl -L https://github.com/protocolbuffers/protobuf/releases/download/v21.5/protoc-21.5-linux-x86_64.zip -o protoc.zip
unzip protoc.zip
rm protoc.zip
# If /usr/local/bin is in your $PATH
ln -s $(pwd)/bin/protoc /usr/local/bin/protoc
cd ../..

cd scripts
npm install ts-proto@1.121.6 --save-dev --save-exact
cd ..

mkdir -p client/src/types/generated

grep cosmos-sdk go.mod

mkdir -p proto/cosmos/base/query/v1beta1
curl https://raw.githubusercontent.com/cosmos/cosmos-sdk/v0.45.4/proto/cosmos/base/query/v1beta1/pagination.proto -o proto/cosmos/base/query/v1beta1/pagination.proto
mkdir -p proto/google/api
curl https://raw.githubusercontent.com/cosmos/cosmos-sdk/v0.45.4/third_party/proto/google/api/annotations.proto -o proto/google/api/annotations.proto
curl https://raw.githubusercontent.com/cosmos/cosmos-sdk/v0.45.4/third_party/proto/google/api/http.proto -o proto/google/api/http.proto
mkdir -p proto/gogoproto
curl https://raw.githubusercontent.com/cosmos/cosmos-sdk/v0.45.4/third_party/proto/gogoproto/gogo.proto -o proto/gogoproto/gogo.proto

cd scripts
ls ../proto/checkers | xargs -I {} protoc \
    --plugin="./node_modules/.bin/protoc-gen-ts_proto" \
    --ts_proto_out="../client/src/types/generated" \
    --proto_path="../proto" \
    --ts_proto_opt="esModuleInterop=true,forceLong=long,useOptionals=messages" \
    checkers/{}

# Or with docker
ls proto/checkers | xargs -I {} docker run --rm \
    -v $(pwd):/checkers \
    -w /checkers/scripts \
    checkers_i \
    protoc \
    --plugin="./node_modules/.bin/protoc-gen-ts_proto" \
    --ts_proto_out="../client/src/types/generated" \
    --proto_path="../proto" \
    --ts_proto_opt="esModuleInterop=true,forceLong=long,useOptionals=messages" \
    checkers/{}

# Rerun protoc file
make gen-protoc-ts
```

## Add leaderboard module

```sh
ignite scaffold module leaderboard --ibc

# store player data
ignite scaffold map playerInfo \
    wonCount:uint lostCount:uint forfeitedCount:uint \
    dateUpdated:string \
    --module leaderboard \
    --no-message

ignite scaffold single board \
    PlayerInfo:PlayerInfo \
    --module leaderboard \
    --no-message

# Regenerate mocks
make mock-expected-keepers

ignite scaffold message updateBoard --module leaderboard

ignite scaffold packet candidate \
    PlayerInfo:PlayerInfo \
    --module leaderboard
```
