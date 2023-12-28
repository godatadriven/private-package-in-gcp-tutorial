# How to register and install your own private Python package in GCP Artifact Registry

For more info, see the [official documentation](https://cloud.google.com/artifact-registry/docs/python/manage-packages).

1. Create a repository in GCP Artifact Registry to store our package:
   1. Navigate to https://console.cloud.google.com/artifacts
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

## Register package with CICD

1. Create a service account
2. Follow the instructions [here](https://cloud.google.com/artifact-registry/docs/python/authentication#sa-key)
3. WIP...
