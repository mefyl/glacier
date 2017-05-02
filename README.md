# AWS Glacier frontend

## Rationale

The AWS CLI offers low level API calls to glacier
functionalities. E.g. to upload an archive, one must split it in parts
following specific size rules, upload parts individually computing
non-trivial tree hashes of each part, computing the final tree hash of
the archive, etc. Moreover, given the potential size of the archive,
resumability is needed, so one must keep track of the upload id, the
parts already uploaded, ...

All of this prompts for a higher level frontend to simplify glacier
operations. While some scripts already exist out there, most of them
are hackish, with mostly hardcoded values (configuration, credentials,
vault ...), still require manual steps like splitting the file in
parts, don't support configurable part size ...

This is my attempt at a clean, generic glacier frontend.

## Suported features

* Multipart archive upload, resumable and abortable.
* Inventory of a vault.
* Deletion of archives.
* Tree hashing of local files, for integrity checking.

## Quickstart

Firstly, create or retrieve an AWS access key and secret key. Create a
vault in the AWS web console if needed.

Launch the multipart upload:

```sh
./glacier upload -k AWS_ACCESS_KEY -s AWS_SECRET_KEY -v VAULT_NAME path/to/file
```

The uploaded archive id and informations will be displayed once
done. The archive description will be the file path as passed. If the
process is interrupted voluntarily or accidentally, you can resume it
with the same command later. Although you need not know this, the
state of the upload is pickled in a '.filename.glacier-upload'
metadata file next to the uploaded file.

If you want to abort your upload, use the same arguments with `abort`:

```sh
./glacier abort -k AWS_ACCESS_KEY -s AWS_SECRET_KEY -v VAULT_NAME path/to/file
```

To list the content of a vault, use:

```sh
./glacier inventory -k AWS_ACCESS_KEY -s AWS_SECRET_KEY -v VAULT_NAME
```

Note that the Glacier job can take hours to prepare the inventory, so
don't be surprised if the command stales for a very long
time. Interrupting the command is handled, if you restart it later it
will reattach to the same inventory job (inventory stay available for
at least 24 hours). In case you do not want to reuse the previous
inventory and force an new one, use `-f`.

To delete an archive, use:

```sh
./glacier delete -k AWS_ACCESS_KEY -s AWS_SECRET_KEY -v VAULT_NAME -a ARCHIVE_ID
```

You can obtain archive ids just after the upload or by running an
inventory of the vault.

To check integrity, you can obtain the tree hash of a file using:

```sh
./glacier treehash path/to/file
```

This should match the treehash returned at the end of an upload and in
inventories.

## TODO

* Downloading archives.
* Creating/deleting vault.
* Uploading parts in parallel.
