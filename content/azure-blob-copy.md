Title: Copy directly from azure blob storage to azure data lake.
Date: 2018-09-26 15:26
Category: Python
Tags: Azure, python
Author: Christian Sloper
Summary: Copy direct from blob storage to data lake


A cool snippet copying files directly from azure blob storage to azure data lake.

Neither the Azure Data Lake client nor the Azure Storage client allows for directly copying files between these two systems even if they are used in parallell.  However, combining the possibility to get a stream from the blob, and the open file function from azure data lake we can copy the blob without downloading it first.

First, get the blob client:

```python

from azure.storage.blob import BlockBlobService

container = "mycontainer"
account_name = secrets["blob_account_name"]
account_key = secrets["blob_key"]
block_blob_service = BlockBlobService(account_name, account_key)

```

Then, get the data lake client:

```python
from azure.datalake.store import core, lib

tenant = config.secrets["tenant"]
resource = config.secrets["datalakeresource"]
client_id = config.secrets["datalakeclientid"]
client_secret = config.secrets["datalakeclientsecret"]

credentials = lib.auth(tenant_id=tenant,
                    client_secret=client_secret,
                    client_id=client_id,
                    resource=resource)

adlfsClient  =  core.AzureDLFileSystem(credentials, store_name=config.secrets["datalakestorename"])
```

Now, we can create the following useful function:

```python
def copy_blob_to_lake(adlfsClient, block_blob_service, blob_name,lake_path):
    with adlfsClient.open(lake_path, 'wb') as file:
        block_blob_service.get_blob_to_stream(container_name=container, blob_name=blob_name, stream=file, max_connections=1)
```