# How to register and install your own private Python package in GCP Artifact Registry

In the following tutorial, we will register and install a private Python package in GCP Artifact Registry. We will do this in two ways: manually and with CICD (Github Actions). In the end, you will be able to install your own private Python package in your Python projects, by adding it to your requirements as you're used to!

For more info, see the [official documentation](https://cloud.google.com/artifact-registry/docs/python/manage-packages). See [this documentation page](https://cloud.google.com/artifact-registry/docs/python/authentication) for more info about authentication with GCP Artifact Registry. See [this blog post](https://xebia.com/blog/an-updated-guide-to-setuptools-and-pyproject-toml/) for more info on how to package your Python code.

## Register and install package manually

1. Create a repository in GCP Artifact Registry to store our package:
   1. Navigate to [Artifact Registry](https://console.cloud.google.com/artifacts)
   2. Click **+ CREATE REPOSITORY**
   3. Give it an appropriate name
   4. Select **Format**: Python
   5. Pick a region near you


2. Make sure you're logged in with the gcloud CLI:

```bash
gcloud auth login
```

3. Create a virtual environment and activate it

```bash
python -m venv .venv
source .venv/bin/activate
```

4. Copy and rename `env.template` to `.env` and set variables:

```bash
PROJECT_ID=<your-project-id>
REPOSITORY=<your-artifact-registry-repository-name>
LOCATION=<the-location-of-your-repository>
```

5. Make environment variables available to your shell.

```bash
set -a  # configure variable assignments to be exported
```
```bash
source .env  # set the variables
```

6. Install the following libraries:

```bash
pip install build  # for building our package
pip install twine  # for uploading the package to GCP Artifact Registry
pip install keyring  # for storing credentials
pip install keyrings.google-artifactregistry-auth  # for storing credentials
```

7. Create a `.pypirc` file in your home directory. Note: this overwrites an existing `.pypirc` file if it exists. If you have an existing `.pypirc` file, you should manually add the following profile to it.

```bash
echo "[distutils]
index-servers =
    ${REPOSITORY}

[${REPOSITORY}]
repository: https://${LOCATION}-python.pkg.dev/${PROJECT_ID}/${REPOSITORY}/" > ~/.pypirc
```

8. Create a `pip.conf` file in your virtual environment:
```bash
echo "[global]
extra-index-url = https://${LOCATION}-python.pkg.dev/${PROJECT_ID}/${REPOSITORY}/simple/" > .venv/pip.conf
```

9. Build your package. This will create a `dist` directory with a `.tar.gz` and `.whl` file in it.

```bash
python -m build
```

10. Upload your package to GCP Artifact Registry:

```bash
python -m twine upload -r ${REPOSITORY} dist/*
```

11. Install your package from GCP Artifact Registry:

```bash
pip install your-own-private-package
```

## Register package with CICD (Github Actions)

1. Fork this Github repository, so you will be able to add secrets to it later on. You can also use your own repository, but you will need to update the CICD configuration accordingly.
1. Navigate to **IAM & Admin** > **Service Accounts** in GCP
2. Create a new service account with the "Artifact Registry Writer" role
3. Add a key to the service account and download it as a JSON file. Never commit this file to your repository!
4. Add the following secrets to your Github repository, under **Settings** > **Secrets and variables** > **Actions** > **Repository secrets**:
   - `PROJECT_ID`: your GCP project ID
   - `LOCATION`: the location of your Artifact Registry repository
   - `REPOSITORY`: the name of your Artifact Registry repository
   - `SA_KEY_BASE64`: the contents of the service account JSON file you downloaded in step 3, base64 encoded. You can use the following command to base64 encode the file: 

```bash
base64 -i <path-to-json-file>
```
6. There are two ways to trigger the provided workflow in `.github/workflows/register-package.yaml`:
   1. Manually, by clicking **Run workflow** in the **Actions** tab of your Github repository
   2. Automatically, by pushing a new tag to your repository. The tag name must start with `v` and be followed by a version number, e.g. `v1.0.0`. You can do this by running the following command:
```bash
git tag v1.0.0
git push origin v1.0.0
```
7. Check the **Actions** tab in your Github repository to see the CICD pipeline in action. You can also check the Artifact Registry repository to see if your package was uploaded successfully.

## Wrap up

Congratulations! You have successfully registered and installed your own private Python package in GCP Artifact Registry. Provided you set up your pip index as outlined in the first section, you can now use this package in your Python projects, by adding it to your requirements as you're used to! 🎉
