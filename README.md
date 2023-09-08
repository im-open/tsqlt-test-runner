# DEPRECATED

This action was deprecated on 2023-09-08 and will no longer receive support or updates.

# tsqlt-test-runner

An Action for running [tSqlt](https://tsqlt.org/) tests against a database. It will generate an xml file with the results of the test run.

## Index <!-- omit in toc -->

- [tsqlt-test-runner](#tsqlt-test-runner)
  - [Inputs](#inputs)
  - [Outputs](#outputs)
  - [Usage Examples](#usage-examples)
  - [Contributing](#contributing)
    - [Incrementing the Version](#incrementing-the-version)
    - [Source Code Changes](#source-code-changes)
    - [Updating the README.md](#updating-the-readmemd)
  - [Code of Conduct](#code-of-conduct)
  - [License](#license)

## Inputs

| Parameter                 | Is Required | Default | Description                                                                                                                                                                                            |
|---------------------------|-------------|---------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `db-server-name`          | true        | N/A     | The database server name.                                                                                                                                                                              |
| `db-server-port`          | false       | 1433    | The port the database server listens on.                                                                                                                                                               |
| `db-name`                 | true        | N/A     | The name of the database to run tests against.                                                                                                                                                         |
| `query-timeout`           | false       | N/A     | An optional setting for the allowed wait time, in seconds, for a statement to execute. If tests sometimes hang, or shouldn't take longer than a certain amount of time, this parameter can be helpful. |
| `use-integrated-security` | true        | false   | Use domain integrated security. If false, a db-username and db-password should be specified. If true, those parameters will be ignored if specified.                                                   |
| `db-username`             | false       | N/A     | The username to use to login to the database. This is required if use-integrated-security is false, otherwise it's optional and will be ignored.                                                       |
| `db-password`             | false       | N/A     | The password for the user logging in to the database. This is required if use-integrated-security is false, otherwise it's optional and will be ignored.                                               |

## Outputs

| Output                          | Description                            |
|---------------------------------|----------------------------------------|
| `test-results-file-path`        | The path to the test results xml file. |
| `total-number-of-tests`         | The total number of tests run.         |
| `total-number-of-test-failures` | The total number of tests that failed. |
| `total-number-of-test-errors`   | The total number of test errors.       |

## Usage Examples

```yml
jobs:
  build-and-test-database:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v3

      - name: Setup Flyway
        uses: im-open/setup-flyway@v1
        with:
          version: 5.1.4

      - name: Create the database
        run: |
          # ...
      - name: Initialize the database
        run: |
          # Do any work needed to initialize the database
          # ...

      # Uses flyway to run migration scripts that will add tSqlt and tests to the database
      - name: Add tSqlt to the database
        uses: im-open/run-flyway-command@v1
        with:
          db-server-name: 'localhost'
          db-server-port: '1433'
          db-name: 'MyDatabase'
          migration-files-path: 'path/to/tsqlt/files'
          flyway-command: 'migrate'
          migration-history-table: 'TestingHistory'
          managed-schemas: 'dbo'
          validate-migrations: 'false'
          use-integrated-security: 'false'
          username: 'sa'
          password: '${{ secrets.MY_SA_PASSWORD }}'

      # Run the tests
      - name: Run tSqlt tests
        id: run-tests
        # You may also reference the major or major.minor version
        uses: im-open/tsqlt-test-runner@v1.1.1
        with:
          db-server-name: 'localhost'
          db-server-port: '1433'
          db-name: 'MyDatabase'
          query-timeout: '120' # 2 minutes
          use-integrated-security: 'false'
          db-username: 'sa'
          db-password: '${{ secrets.MY_SA_PASSWORD }}'

      - name: Print outputs
        shell: bash
        run: |
          echo "Path to test results: ${{ steps.run-tests.outputs.test-results-file-path }}"
          echo "Total number of tests: ${{ steps.run-tests.outputs.total-number-of-tests }}"
          echo "Number of failed tests: ${{ steps.run-tests.outputs.total-number-of-test-failures }}"
          echo "Number of test errors: ${{ steps.run-tests.outputs.total-number-of-test-errors }}"
```

## Contributing

When creating PRs, please review the following guidelines:

- [ ] The action code does not contain sensitive information.
- [ ] At least one of the commit messages contains the appropriate `+semver:` keywords listed under [Incrementing the Version] for major and minor increments.
- [ ] The README.md has been updated with the latest version of the action.  See [Updating the README.md] for details.

### Incrementing the Version

This repo uses [git-version-lite] in its workflows to examine commit messages to determine whether to perform a major, minor or patch increment on merge if [source code] changes have been made.  The following table provides the fragment that should be included in a commit message to active different increment strategies.

| Increment Type | Commit Message Fragment                     |
|----------------|---------------------------------------------|
| major          | +semver:breaking                            |
| major          | +semver:major                               |
| minor          | +semver:feature                             |
| minor          | +semver:minor                               |
| patch          | *default increment type, no comment needed* |

### Source Code Changes

The files and directories that are considered source code are listed in the `files-with-code` and `dirs-with-code` arguments in both the [build-and-review-pr] and [increment-version-on-merge] workflows.  

If a PR contains source code changes, the README.md should be updated with the latest action version.  The [build-and-review-pr] workflow will ensure these steps are performed when they are required.  The workflow will provide instructions for completing these steps if the PR Author does not initially complete them.

If a PR consists solely of non-source code changes like changes to the `README.md` or workflows under `./.github/workflows`, version updates do not need to be performed.

### Updating the README.md

If changes are made to the action's [source code], the [usage examples] section of this file should be updated with the next version of the action.  Each instance of this action should be updated.  This helps users know what the latest tag is without having to navigate to the Tags page of the repository.  See [Incrementing the Version] for details on how to determine what the next version will be or consult the first workflow run for the PR which will also calculate the next version.

## Code of Conduct

This project has adopted the [im-open's Code of Conduct](https://github.com/im-open/.github/blob/main/CODE_OF_CONDUCT.md).

## License

Copyright &copy; 2023, Extend Health, LLC. Code released under the [MIT license](LICENSE).

<!-- Links -->
[Incrementing the Version]: #incrementing-the-version
[Updating the README.md]: #updating-the-readmemd
[source code]: #source-code-changes
[usage examples]: #usage-examples
[build-and-review-pr]: ./.github/workflows/build-and-review-pr.yml
[increment-version-on-merge]: ./.github/workflows/increment-version-on-merge.yml
[git-version-lite]: https://github.com/im-open/git-version-lite
