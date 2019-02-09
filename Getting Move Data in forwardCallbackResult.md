# Getting and Processing Move Data in forwardCallbackResult

While your game will be more complex, and certainly have more substance to your moves, our aim here isn't to examine "Mover" per se. Instead, we'll look at the general structure which will still largely be the same in your game. In order to do that, we'll break down and examine each piece of the puzzle in order to gain a greater understanding. Should anything seem trivial, simply skip ahead. 

Moves submitted to the blockchain are processed in the forwardCallbackResult method. This is its signature:

	public static string forwardCallbackResult(string oldState, string blockData, string undoData, out string newData)

Moves are in `blockData`. They can be deserialised as follows:

	dynamic blockDataS = JsonConvert.DeserializeObject(blockData);

However, you can process them however you wish. 

This is a move from the Mover example game:

	{{
	  "block": {
	    "hash": "6d8071aa2e91e13004d9cc185e3a51a1975f33f190b171ce3d638ae17fdb1154",
	    "height": 244774,
	    "parent": "e58f9eed7447fa638d75e43eebce740579c7236938d7eaf099c016e4884056d5",
	    "rngseed": "42aa6588de84aabc952ee7815bf04b1a8953bea1877fa1b11b18b40aefdccd28",
	    "timestamp": 1539182355
	  },
	  "moves": [
	    {
	      "move": {
			  "d": "u",
			  "n": 10
	      },
	      "name": "domob",
	      "out": {
		  "CJHMmfxKEypZd9oF5Ui9cSgWD7eaUC6cGT": 4.732778
	      },
	      "txid": "83747d1e69e264644f425bc2dae3bedca6d72274a0f2f82b00d9a0a70714c2be"
	    }
	  ],
	  "reqtoken": "cf8efee5419ca175f67f8318a1977160"
	}}

The `block` and `reqtoken` fields are fixed. The `moves` field is an array of moves. 

Each `moves` array has move data. The `name` is the `p/` name of the player. You'll need this to process the player's move(s). The `out` and `txid` are highly unlikely to be useful to process moves in a game. However, they could be useful for verification purposes in some applications. 

The `move` in the example above is:


	"move": {
		"d": "u",
		"n": 10
		}

In Mover, the "d" is for the direction, and the "n" is for the number of steps to take. Your game's moves will be more complex than this simple example. 

So `blockDataS["moves"]` yields an array of individual moves. 

A `blockDataS` with no moves would look similar to the following.

	{
		"block":
			{
				"hash":"44dd0655875c7538adaa47b7c66cb57e861397341e06c63715285916f9d026d8",
				"height":555556,
				"parent":"ce6a6ae43103db943a74294b90906de9bb873d602f2881ddb3eb7a9f0e626312",
				"rngseed":"9f73fe227ac8ac4d8fe17721c554ea631792e270019dfb5262f73d3750ee8684",
				"timestamp":1548967987
			},
		"moves":[],
		"reqtoken":"83433d97299a99efad71661afb54bb04"
	}






{[
  {
    "move": {
      "d": "u",
      "n": 10
    },
    "name": "domob",
    "out": {
      "CJHMmfxKEypZd9oF5Ui9cSgWD7eaUC6cGT": 4.732778
    },
    "txid": "83747d1e69e264644f425bc2dae3bedca6d72274a0f2f82b00d9a0a70714c2be"
  }
]}

blockData:
"{\"block\":{\"hash\":\"44dd0655875c7538adaa47b7c66cb57e861397341e06c63715285916f9d026d8\",\"height\":555556,\"parent\":\"ce6a6ae43103db943a74294b90906de9bb873d602f2881ddb3eb7a9f0e626312\",\"rngseed\":\"9f73fe227ac8ac4d8fe17721c554ea631792e270019dfb5262f73d3750ee8684\",\"timestamp\":1548967987},\"moves\":[],\"reqtoken\":\"7c39c7f470fe1fda739ace2d3b8620f5\"}"


JObject obj = JsonConvert.DeserializeObject<JObject>(m["move"].ToString());
{{
  "m": "HELLO WORLD!"
}}



public class BlockData
{
	[JsonProperty(PropertyName = "block")]
	public Block Block { get; set; }

}

public class Block
{
	[JsonProperty(PropertyName = "hash")]
	public string Hash { get; set; }
	[JsonProperty(PropertyName = "height")]
	public string Height { get; set; }
	[JsonProperty(PropertyName = "parent")]
	public string Parent { get; set; }
	[JsonProperty(PropertyName = "rngseed")]
	public string Rngseed { get; set; }
	[JsonProperty(PropertyName = "timestamp")]
	public string Timestamp { get; set; }
}

public class Moves
{

}








{{
  "block": {
    "hash": "44dd0655875c7538adaa47b7c66cb57e861397341e06c63715285916f9d026d8",
    "height": 555556,
    "parent": "ce6a6ae43103db943a74294b90906de9bb873d602f2881ddb3eb7a9f0e626312",
    "rngseed": "9f73fe227ac8ac4d8fe17721c554ea631792e270019dfb5262f73d3750ee8684",
    "timestamp": 1548967987
  },
  "moves": [],
  "reqtoken": "11352f3c647924c9fcfec8e0c91fb7ec"
}}


{{
  "block": {
    "hash": "44dd0655875c7538adaa47b7c66cb57e861397341e06c63715285916f9d026d8",
    "height": 555556,
    "parent": "ce6a6ae43103db943a74294b90906de9bb873d602f2881ddb3eb7a9f0e626312",
    "rngseed": "9f73fe227ac8ac4d8fe17721c554ea631792e270019dfb5262f73d3750ee8684",
    "timestamp": 1548967987
  },
  "moves": [],
  "reqtoken": "11352f3c647924c9fcfec8e0c91fb7ec"
}}
