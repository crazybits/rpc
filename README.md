# go-steem/rpc

Golang RPC client library for [Steem](https://steem.io).

## Usage

```go
import "github.com/go-steem/rpc"
```

2. Use `rpc.Dial` to get an RPC client.
3. PROFIT!

## Example

This is just a code snippet. Please check the `examples` directory
for more complete and ready to use examples.

```go
// Instantiate a new client.
client, _ := rpc.Dial("ws://localhost:8090")
defer client.Close()

// Call "get_config".
config, _ := client.GetConfig()

// Start processing blocks.
lastBlock := 1800000
for {
	// Call "get_dynamic_global_properties".
	props, _ := client.GetDynamicGlobalProperties()

	for props.LastIrreversibleBlockNum-lastBlock > 0 {
		// Call "get_block".
		block, _ := client.GetBlock(lastBlock)

		// Process the transactions.
		for _, tx := range block.Transactions {
			for _, op := range tx.Operations {
				switch body := op.Body.(type) {
					// Comment operation.
					case *rpc.CommentOperations:
						content, _ := client.GetContent(body.Author, body.Permlink)
						fmt.Printf("COMMENT @%v %v\n", content.Author, content.URL)

					// Vote operation.
					case *rpc.VoteOperation:
						fmt.Printf("@%v voted for @%v/%v\n", body.Voter, body.Author, body.Permlink)

					// You can add more cases, it depends on what
					// operations you actually need to process.
				}
			}
		}

		lastBlock++
	}

	time.Sleep(time.Duration(config.SteemitBlockInterval) * time.Second)
}
```

## Package Organisation

Once you create a `Client` object by using `Dial`, you can start calling its methods,
which correspond to the methods exported via `steemd`'s RPC endpoint.

There are two versions for every method. The regular method and the raw method.
The difference is that the raw method returns `*json.RawMessage`, so it is not
trying to unmarshall the response into the right object. The reason for this
distinction is to be able to start using the `rpc` package even though not all
methods are specified properly yet.

## Status

This package is still under rapid development and it is by no means complete.
There is no promise considering API stability.

For now there are raw methods specified for most of the RPC methods available,
but not much has been tested really. Please report any bugs you find. Feel free
to send a pull request as well.

## License

MIT, see the `LICENSE` file.
