---
title: NeRu Secrets
description: This documentation describes the NeRu Secrets
meta_title: NeRu Secrets
---

# NeRu Secrets

NeRu Secrets allow you to store sensitive information for use in your projects. Secrets are managed through the NeRu CLI.

## Creating a Secret

To create a secret, you can use the NeRu CLI secrets create command.

```sh
neru secrets create --name <name> --value <value>
```

So to create a secret `foo` with the value `bar`:

```sh
neru secrets create --name foo --value bar
```

You can also add files as secrets:

```sh
neru secrets create --name <name> --file <path/to/file>
```

## Accessing your Secrets

When secrets are injected into your instance, they are converted into upper snake case and prefixed with `NERU_SECRET`. So for the earlier example of `foo`, it will be available in your instance on the environment as `NERU_SECRET_FOO`.

## Updating Secrets

Updating secrets works similarly to creating secrets, but to update you use `neru secrets update`:

```sh
neru secrets update --name <name> --value <changed-value>
```

To update the `foo` example:

```sh
neru secrets update --name foo --value baz
```

## Removing Secrets

Secrets can be removed using `neru secrets remove`:

```sh
neru secrets remove <secret_name>
```

To remove the `foo` example:

```sh
neru secrets remove foo
```