# tsqlt-test-runner

An Action for running [tSqlt](https://tsqlt.org/) tests against a database. It will generate an xml file with the results of the test run.

## Index

- [tsqlt-test-runner](#tsqlt-test-runner)
  - [Index](#index)
  - [Inputs](#inputs)
  - [Outputs](#outputs)
  - [Example](#example)
  - [Contributing](#contributing)
    - [Incrementing the Version](#incrementing-the-version)
  - [Code of Conduct](#code-of-conduct)
  - [License](#license)

## Inputs

| Parameter                 | Is Required | Default | Description                                                                                                                                                                                            |
| ------------------------- | ----------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `db-server-name`          | true        | N/A     | The database server name.                                                                                                                                                                              |
| `db-server-port`          | false       | 1433    | The port the database server listens on.                                                                                                                                                               |
| `db-name`                 | true        | N/A     | The name of the database to run tests against.                                                                                                                                                         |
| `query-timeout`           | false       | N/A     | An optional setting for the allowed wait time, in seconds, for a statement to execute. If tests sometimes hang, or shouldn't take longer than a certain amount of time, this parameter can be helpful. |
| `use-integrated-security` | true        | false   | Use domain integrated security. If false, a db-username and db-password should be specified. If true, those parameters will be ignored if specified.                                                   |
| `db-username`             | false       | N/A     | The username to use to login to the database. This is required if use-integrated-security is false, otherwise it's optional and will be ignored.                                                       |
| `db-password`             | false       | N/A     | The password for the user logging in to the database. This is required if use-integrated-security is false, otherwise it's optional and will be ignored.                                               |

## Outputs

| Output                          | Description                            |
| ------------------------------- | -------------------------------------- |
| `test-results-file-path`        | The path to the test results xml file. |
| `total-number-of-tests`         | The total number of tests run.         |
| `total-number-of-test-failures` | The total number of tests that failed. |
| `total-number-of-test-errors`   | The total number of test errors.       |

## Example

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

When creating new PRs please ensure:

1. For major or minor changes, at least one of the commit messages contains the appropriate `+semver:` keywords listed under [Incrementing the Version](#incrementing-the-version).
1. The action code does not contain sensitive information.

When a pull request is created and there are changes to code-specific files and folders, the `auto-update-readme` workflow will run.  The workflow will update the action-examples in the README.md if they have not been updated manually by the PR author. The following files and folders contain action code and will trigger the automatic updates:

- `action.yml`
- `src/**`

There may be some instances where the bot does not have permission to push changes back to the branch though so this step should be done manually for those branches. See [Incrementing the Version](#incrementing-the-version) for more details.

### Incrementing the Version

The `auto-update-readme` and PR merge workflows will use the strategies below to determine what the next version will be.  If the `auto-update-readme` workflow was not able to automatically update the README.md action-examples with the next version, the README.md should be updated manually as part of the PR using that calculated version.

This action uses [git-version-lite] to examine commit messages to determine whether to perform a major, minor or patch increment on merge. The following table provides the fragment that should be included in a commit message to active different increment strategies.
| Increment Type | Commit Message Fragment |
| -------------- | ------------------------------------------- |
| major | +semver:breaking |
| major | +semver:major |
| minor | +semver:feature |
| minor | +semver:minor |
| patch | *default increment type, no comment needed* |

## Code of Conduct

This project has adopted the [im-open's Code of Conduct](https://github.com/im-open/.github/blob/master/CODE_OF_CONDUCT.md).

## License

Copyright &copy; 2021, Extend Health, LLC. Code released under the [MIT license](LICENSE).

[git-version-lite]: https://github.com/im-open/git-version-lite
