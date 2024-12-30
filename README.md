# AO Keystore Action

This GitHub Action is designed to create a keystore file from a Base64-encoded string and validate it. It can be used to
ensure that the keystore and its contents (such as aliases and passwords) are correct before using it in your Android
build process.

## Inputs

| Input Name          | Description                                                            | Required | Default Value  |
|---------------------|------------------------------------------------------------------------|----------|----------------|
| `base64-keystore`   | The Base64-encoded content of the keystore file.                       | ✅        |                |
| `keystore-password` | Password of the keystore file.                                         | ✅        |                |
| `keystore-path`     | The location and file where the decoded keystore file will be written. | ❌        | `keystore.jks` |
| `key-alias`         | The alias of the key in the keystore.                                  | ✅        |                |
| `key-password`      | The password for the keystore and key alias.                           | ✅        |                |

## Outputs

| Output Name       | Description                                         |
|-------------------|-----------------------------------------------------|
| `isKeystoreValid` | `true` if the keystore is valid, `false` otherwise. |

## Example Usage

Here’s an example of how to use this action in your GitHub Actions workflow:

```yaml
name: Test AO Keystore Action

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Create Dummy Keystore (for testing)
        run: |
          # Generate the keystore file
          keytool -genkeypair -v -keystore test-keystore.jks -keyalg RSA -keysize 2048 -validity 1 -storepass changeit -keypass changeit -dname "CN=Test, OU=Test, O=Test, L=Test, ST=Test, C=US" -alias test-alias
          
          # Convert the keystore file to Base64 and save it to the environment variable
          KEYS=$(cat test-keystore.jks | base64 | tr -d '\n')
          echo "base64-keystore=$KEYS" >> $GITHUB_ENV
        continue-on-error: true

      - name: Run AO Keystore Action
        id: validate-keystore
        uses: ajaxer-org/keystore-action@development  # Referencing the action
        with:
          base64-keystore: ${{ env.base64-keystore }}
          keystore-password: changeit
          keystore-path: keystore.jks
          key-alias: test-alias
          key-password: changeit

      - name: Validate Output
        if: env.isKeystoreValid != 'true'
        run: |
          echo "Keystore validation failed!"
          exit 1

      - name: Success Message
        if: env.isKeystoreValid == 'true'
        run: echo "Keystore validation passed!"
