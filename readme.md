# API Instructions

## Local Development

#### Create a bucket for dev uploads
	aws s3api create-bucket --bucket singledigit-dev-uploads --profile ej

#### Packaging your service
	aws cloudformation package --template-file service.yml \
		--s3-bucket singledigit-dev-uploads \
		--output-template-file ./.sam/package-service.yml \
		--profile ej
			
#### Deploying your service
	aws cloudformation deploy --template-file ./.sam/package-service.yml \
		--stack-name pipeline-service \
		--capabilities CAPABILITY_IAM \
		--profile ej
			
#### Deploy your function
	zip -j ./.webpack/index.zip ./.webpack/* && \
	aws lambda update-function-code \
		--function-name pipeline-service-index \
		--zip-file fileb://./.webpack/index.zip \
		--profile ej
			
#### Invoke your function
	aws lambda invoke --function-name pipeline-service-index output.json --profile singledigit && \
		cat output.json
		
##	CICD
#### Create a bucket for CFT files in new environment
	aws s3api create-bucket --bucket singledigit-demo-api-cft --profile singledigit

#### Copy CFT files to bucket
	aws s3 sync ./cft s3://singledigit-demo-api-cft --profile singledigit	
#### Creating the CICD pipeline
	aws cloudformation create-stack \
		--profile singledigit \
		--stack-name pipeline-service-cicd \
       --template-url=https://s3.amazonaws.com/singledigit-demo-api-cft/cicd.yml \
       --capabilities=CAPABILITY_NAMED_IAM \
       --parameters ParameterKey=Service,ParameterValue=pipeline-service \
       ParameterKey=GitHubOwner,ParameterValue=singledigit \
       ParameterKey=Repo,ParameterValue=pipeline-api \
       ParameterKey=Branch,ParameterValue=master \
       ParameterKey=Token,ParameterValue=