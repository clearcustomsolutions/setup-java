# 0. V2 setup-java

Date: 2020-08-24

Status: Proposed

# Context

- The `v1` version of `setup-java` downloads and installs Zulu builds of OpenJDK. There is a huge ask from customers to offer AdoptOpenJDK builds of OpenJDK: https://github.com/actions/setup-java/issues/13

- Zulu and AdoptOpenJDK aren't the only distributions of Java though. Other providers include Oracle OpenJDK or Amazon Corretto and ideally it would be nice to support downloading Java from all providers.

- GitHub Actions virtual environments install and default to AdoptOpenJDK builds of OpenJDK. `setup-java` should align to the same behavior.

# Decision

## New input

A new input parameter (titled `distribution`) will be added to `setup-java` that will allow users to specify the distribution that they would like to download

```yaml
  with:
    java-version: '11'
    distribution: adoptopenjdk
```

## Default Behavior

If no `distribution` parameter is provided, the default value will be `adoptopenjdk`. The action will first attempt to use pre-installed versions of AdoptOpenJDK builds of OpenJDK on GitHub hosted runners. If a specific version is not found, the action will then download and install the correct version.

## Extensibility & Documentation

`setup-java` should be structured in such a way that will allow the open source community to easily add support for extra distributions.

Existing code will be restructured so that distribution specific code will be easily separated. Currently the core download logic is in a single file, `installer.ts`. This file will be split up and abstracted out so that there will be no vendor specified logic. Each distribution will have it's own file under `src/distributions` that will contain the core setup logic for a specific distribution. 

```yaml
 ∟ src/
    installer.ts # core installer logic
    distributions/
        ∟ adoptOpenJDK.ts # adoptOpenJDK specific logic
        ∟ zulu.ts # zulu specific logic
        ∟ # future distributions will be added here 
```

Each specific distribution can have minute differences or other information that should be documented. There will be a directory for distribution specific information under `docs/`. The main `README.md` will document all available distributions and link to specific documentation. The main `README.md` should not contain distribution specific information.

```yaml
 ∟ docs/
    distributions/    
        ∟ adoptOpenJDK.md # adoptOpenJDK specific info
        ∟ zulu.md # zulu specific info
        ∟ # future distributions will be added here 

```

The contribution doc (`CONTRIBUTING.md`) will describe how a new distribution should be added and how everything should be structured.

## v2-preview

There will be a `v2-preview` branch that will be created for development and testing. Any changes will first be merged into `v2-preview` branch. After a period of testing & verification, the `v2-preview` branch will be merged into the `main` branch and a `v2` tag will be created. Any [GitHub public documentation](https://docs.github.com/en/actions/language-and-framework-guides/github-actions-for-java) and [starter workflows](https://github.com/actions/starter-workflows) that mention `setup-java` will then be updated to use `v2` instead of `v1`.

## Goals & Anti-Goals

The main focus of the `v2` version of `setup-java` will be to add support for adoptOpenJDK builds of openJDK in addition to Zulu builds of openJDK. In addition, extensibility will be a priority so that other distributions can be added in the future.

The `setup-java` action has some logic that creates a `settings.xml` file so that it is easier to publish packages. Any improvements or modifications to this logic or anything Gradle/Maven specific will be avoided during the development of the `v2-preview`.

# Consequences

- Users will have more flexibility and the freedom to choose a specific distribution that they would like (AdoptOpenJDK builds of OpenJDK in addition or Zulu builds of OpenJDK)
- `setup-java` will be structured in such a way that will allow for more distributions to be easily added in the future
- A large subset of users pin to `@main` or `@master` instead of to a specific version (`v1`). By introducing a breaking change by switching from `Zulu` to `AdoptOpenJDK` as the default, certain existing workflows for users might break
- Higher maintenance and support burden moving forward