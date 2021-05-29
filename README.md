# lndgrpc
A python grpc client for LND (Lightning Network Daemon) ⚡⚡⚡

This is a wrapper around the default grpc interface that handles setting up credentials (including macaroons). An async client is also available to do fun async stuff like listening for invoices in the background. 

## Dependencies
Python 2.7, 3.4+
Note: the async client is only available for Python 3.5+

## Installation
```bash
$ pip install lndgrpc
```

## Basic Usage
The api mirrors the underlying lnd grpc api (http://api.lightning.community/) but methods will be in pep8 style. ie. `.GetInfo()` becomes `.get_info()`.

```python
from lndgrpc import LNDClient

# pass in the ip-address with RPC port and network ('mainnet', 'testnet', 'simnet')
# the client defaults to 127.0.0.1:10009 and mainnet if no args provided
lnd = LNDClient("127.0.0.1:10009", network='simnet')

lnd.get_info()

print('Listening for invoices...')
for invoice in lnd.subscribe_invoices():
    print(invoice)
```

### Async

```python
import asyncio
from lndgrpc import AsyncLNDClient

async_lnd = AsyncLNDClient()

async def subscribe_invoices():
    print('Listening for invoices...')
    async for invoice in async_lnd.subscribe_invoices():
        print(invoice)

async def get_info():
    while True:
        info = await async_lnd.get_info()
        print(info)
        await asyncio.sleep(5)

async def run():
    coros = [subscribe_invoices(), get_info()]
    await asyncio.gather(*coros)

loop = asyncio.get_event_loop()
loop.run_until_complete(run())
```

### Specifying Macaroon/Cert files
By default the client will attempt to lookup the `readonly.macaron` and `tls.cert` files in the mainnet directory. 
However if you want to specify a different macaroon or different path you can pass in the filepath explicitly.

```python
lnd = LNDClient(macaroon_filepath='~/.lnd/invoice.macaroon', cert_filepath='path/to/tls.cert')
```

#### Admin macaroon
Use the admin macaroon to perform write actions (ie. creating invoices, creating new addresses)

```python
lnd = LNDClient(admin=True)
lnd.add_invoice(2000)
```

### Generating Proto Files
```bash
virtualenv lnd
source lnd/bin/activate
pip install grpcio grpcio-tools googleapis-common-protos
git clone https://github.com/googleapis/googleapis.git
git clone https://github.com/lightningnetwork/lnd.git
# python -m grpc_tools.protoc --proto_path=googleapis:. --python_out=. --grpc_python_out=. rpc.proto
```

```python
from pathlib import Path
import shutil

for proto in list(Path("lnd/lnrpc").rglob("*.proto")):
    shutil.copy(proto,Path.cwd())

for proto in list(Path(".").rglob("*.proto")):
    sh.python("-m","grpc_tools.protoc","--proto_path=.","--python_out=.","--grpc_python_out=.", str(proto))
```

Last Step:
In File: verrpc_pb2_grpc.py
Change:
import verrpc_pb2 as verrpc__pb2
To:
from lndgrpc import verrpc_pb2 as verrpc__pb2

## Deploy to Test-PyPi
```bash
poetry build
twine check dist/*
twine upload --repository-url https://test.pypi.org/legacy/ dist/*
```