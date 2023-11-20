# GitHub Actions: Running a single step inside a container

It is possible to run a single step inside a Docker like so

```yaml
- name: Build book
  uses: docker://michaelfbryan/mdbook-docker-image:v0.4.7
```

However, this will use the default `ENTRYPOINT` or run whatever `CMD` points to.
The [official documentation][gha] is not too exhaustive on this.

## ENTRYPOINT

If the container defines an `ENTRYPOINT` additional arguments can be passed
using `args`.

```yaml
- name: Build book
  uses: docker://michaelfbryan/mdbook-docker-image:v0.4.7
  with:
    args: build
```

## CMD

If the container does not have an `ENTRYPOINT` it is possible to pass a command
line to the container which will then be executed as is.

```yaml
- name: Greet
  uses: docker://ubuntu:jammy
  with:
    args: /bin/sh -c "echo Hello"
```

## Notes

- The command runs with the privileges of whatever user is set up inside the
  container (usually root).  This can lead to permission problems when accessing
  files created inside the container in a subsequent step.

- TODO: It should be tested how complex the command passed via `args` can be and
  whether a folded scalar (`>`) or even a literal scalar (`|`) can be used to
  supply multi-line commands.

[gha]: https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action#creating-an-action-metadata-file
