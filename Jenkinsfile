pipeline { 

	agent any 
	options { skipDefaultCheckout() } 
	
	stages { 
		stage('Get Code') { 
			steps { 
				git branch: 'master', url: 'https://github.com/josuandiamunoz/ToDosJosu.git'
			} 
		} 
		
		stage('Deploy') { 
			steps { 
				sh ''' 
					sam build 
					sam deploy --config-env production --debug --no-confirm-changeset --no-fail-on-empty-changeset 
				''' 
			} 
		} 
		stage('Rest Test') {
			steps {
			
				script {
					def BASE_URL = sh( script: "aws cloudformation describe-stacks --stack-name todo-list-aws-production --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text",
						returnStdout: true).trim()
					withEnv(["BASE_URL=${BASE_URL}"]) {
						sh '''
							set -e  # cualquier fallo detiene el script y marca el stage como failed				
							first_todo_id=$(curl -s -f "$BASE_URL/todos" | jq -r '.[0].id')
							echo
							if [ -z "$first_todo_id" ] || [ "$first_todo_id" == "null" ]; then
								echo "⚠️ No TODOs found"
							else
								curl -s -f "$BASE_URL/todos/$first_todo_id"
								echo
							fi

						'''
					}
					
				}
			}
		} 
		
	}
}