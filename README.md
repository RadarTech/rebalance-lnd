# rebalance-lnd

Easily rebalance channels on your LND node.

Reallocate your local funds from one channel to another by creating and fulfilling payments to yourself on circular paths through the network.

Supports both Bitcoin and Litecoin.

## Installation

This script needs an active lnd (https://github.com/lightningnetwork/lnd) instance running.
You need to have admin rights to control this node.
By default this script connects to `localhost:10009`, using the macaroon file in `~/.lnd/data/chain/bitcoin/mainnet/admin.macaroon`.
If you need to change this, please have a look at the output of `python rebalance.py --help`.

This package requires Python3.
The other dependencies can be installed by running:

```
$ pip install -r requirements.txt
```

If you prefer `pipenv`, install by running:

```
pipenv install
```

## Usage

### Command Line Arguments

```
usage: rebalance.py [-h] [--macaroon MACAROON] [--lnddir LNDDIR] [--port PORT]
                    [--host HOST] [-r RATIO] [-l] [-o | -i] [-f CHANNEL]
                    [-t CHANNEL] [-a AMOUNT]

optional arguments:
  -h, --help            show this help message and exit
  --macaroon MACAROON
  --lnddir LNDDIR
  --port PORT           Port for gRPC
  --host HOST           Host IP address for gRPC
  --chain CHAIN         bitcoin or litecoin
  -r RATIO, --ratio RATIO
                        ratio for channel imbalance between 1 and 50%, eg. 45
                        for 45%

list candidates:
  Show the unbalanced channels.

  -l, --list-candidates
                        list candidate channels for rebalance
  -o, --outgoing        lists channels with more than 50% of the funds on the
                        local side
  -i, --incoming        (default) lists channels with more than 50% of the
                        funds on the remote side
  -o, --outgoing        lists channels with more than 50% of the funds on the
                        local side
  -i, --incoming        (default) lists channels with more than 50% of the
                        funds on the remote side

rebalance:
  Rebalance a channel. You need to specify at least the 'to' channel (-t).

  -f CHANNEL, --from CHANNEL
                        channel ID of the outgoing channel (funds will be
                        taken from this channel)
  -t CHANNEL, --to CHANNEL
                        channel ID of the incoming channel (funds will be sent
                        to this channel). You may also use the index as shown
                        in the incoming candidate list (-l -i).
  -a AMOUNT, --amount AMOUNT
                        Amount of the rebalance, in satoshis. If not
                        specified, the amount computed for a perfect rebalance
                        will be used (up to the maximum of 4,294,967 satoshis)
```

### List Channels

Run `rebalance.py -l -i` (or just `rebalance.py -l`) to see a list of channels which can be rebalanced.
This list contains channels where less than 50% of the funds are on the local side of the channel.
When rebalancing is performed, self-payments will arrive via these channels, increasing their local balance.

Run `rebalance.py -l -o` to see a list of channels where less than 50% of the funds are on the remote side of the channel.
When rebalancing is performed, self-payments will depart via this channels, increase their remote balance.

Use `-r/--ratio` to configure the cutoff ratio (default is 50%).
For instance:
- `{-r 50} {-i} -l` will return channels with less than 50% of funds on the local side
- `-r 45 -l` will return channels with less than 45% of funds on the local side
- `-r 30 -l` will return channels with less than 30% of funds on the local side


sensitivity ratio (default is 50%).

As an example the following indicates a channel with around 17.7% of the funds on the local side:

```
(23) Channel ID:  123[...]456
Pubkey:           012345[...]abcdef
Local ratio:      0.176
Capacity:         5,000,000
Remote balance:   4,110,320
Local balance:    883,364
Amount for 50-50: 1,613,478
|=====                       |
```

By sending 1,613,478 satoshis to yourself using this channel, 50:50 local:remote ratio can be achieved.
This number is shown as "Amount for 50-50".

The last line shows a graphical representation of the channel. 
The total width is determined by the channel's capacity, where a channel with maximum capacity (16,777,215 satoshis)
occupies the full width of your terminal.
The bar (`=`) indicates the funds on the local side of the channel.

The number next to the channel ID (23 in the example) can be used to directly reference this channel.

### Rebalancing A Channel

To rebalance a channel, run `rebalance.py --to <id>` or `rebalance.py -t <id>`.
`<id>` can be the 18-digit channel ID or the channel number as specified in `rebalance.py -l`.
Funds are sent *to* the channel specified with `-t`, increasing its local balance.
Optionally, specify the channel to transfer funds *from* using `-f`.
Optionally, specify the amount (in satoshis) to send in the rebalance payment using `-a`.
If an amount is not specified, the lesser of the amount needed to fully rebalance the channel OR the protocol maximum payment size (currently 4,294,967 satoshis) will be sent.

`rebalance.py -t 123[...]456 -a 1613478`

It is also possible to indicate the `--to/-t` channel by the number shown next to the channel ID (23 in the example).

`rebalance.py -t 23 -a 1613478`

### Examples

To move funds into channel 619899158240231424 to increase its local balance to 50% (the amount to move and the channel to source funds will be automatically selected):

```
rebalance.py -t 619899158240231424
```

To move 10,000 satoshis into channel 619899158240231424:

```
rebalance.py -t 619899158240231424 -a 10000
```

To move 10,000 satoshis from channel 610092614007521280 to channel 619899158240231424:

```
rebalance.py -t 619899158240231424 -f 619899158240231424 -a 10000
```

To list unbalanced channels with less than 45% remote balance in LND running on Litecoin at port 10022 with `~/.lnd-ltc` as the LND configuration directory:

```
rebalance.py --chain litecoin --macaroon ~/.lnd-ltc/data/chain/litecoin/mainnet/admin.macaroon --lnddir ~/.lnd-ltc/ --port 10022 -l -o -r 45
```

## Contributing

Contributions of issues, suggestions, and code are all appreciated!

Submit issues and pull requests on https://github.com/radartech/rebalance-lnd/

Check out our development tools, Lightning wiki an newsletter, community resources, and Lightning onboarding guide at https://ion.radar.tech.
