# **CodeArtifact - Hands On**

## Create a CodeArtifact repository

Configure your pypi client using this AWS CLI CodeArtifact command (login authorization expires in 12 hours)

`aws codeartifact login --tool pip --repository DemoRepository --domain my-company --domain-owner ... --region us-east-1`<br />

Manual setup: pulling from your repository with pip<br />
1. Fetch a new authorization token using the AWS CLI
`export CODEARTIFACT_AUTH_TOKEN=$(aws codeartifact get-authorization-token --domain -my-company --domain-owner ... --region us-east-1 --query authorizationToken --output text)`<br /><br />
2. Set the CodeArtifact registry URL and credentials using pip config<br />
`pip config set global.index-url https://aws:$CODEARTIFACT_AUTH_TOKEN@my-company-....d.codeartifact.us-east-1.amazonaws.com/pypi/DemoRepository/simple/`