pipeline { 

	agent any 
	options { skipDefaultCheckout() } 
	
	stages { 
		stage('Get Code') { 
			steps { 
				git branch: 'develop', url: 'https://github.com/josuandiamunoz/ToDosJosu.git'
			} 
		} 
		stage('Static tests') { 
			steps { 
				sh ''' flake8 --exit-zero --format=pylint app >flake8.out ''' 
				sh ''' bandit -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}" || true ''' 
				recordIssues tools: [flake8(pattern: 'flake8.out')]
				recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')] 
			} 
		} 
		
		stage('Deploy') { 
			steps { 
				sh ''' 
					sam build 
					sam deploy --config-env staging --debug --no-confirm-changeset --no-fail-on-empty-changeset 
				''' 
			} 
		} 
		stage('Rest Test') {
			steps {
				sh '''
					set -e  # cualquier fallo detiene el script y marca el stage como failed

					BASE_URL="https://dan0vt5gxf.execute-api.us-east-1.amazonaws.com/Stage"

					echo "➡️ Creating TODO"
					CREATE_RESPONSE=$(curl -s -f -X POST "$BASE_URL/todos" \
					  -H "Content-Type: application/json" \
					  -d '{"text":"Learn Serverless Test"}')

					echo "Response: $CREATE_RESPONSE"

					TODO_ID=$(echo "$CREATE_RESPONSE" | jq -r '.body | fromjson | .id')

					if [ -z "$TODO_ID" ] || [ "$TODO_ID" = "null" ]; then
					  echo "❌ No ID returned from POST"
					  exit 1
					fi
					
					echo "✅ Created TODO with id: $TODO_ID"

					echo "➡️ Listing TODOs"
					curl -s -f "$BASE_URL/todos" > /dev/null
					echo "✅ List OK"

					echo "➡️ Getting TODO by ID"
					curl -s -f "$BASE_URL/todos/$TODO_ID" > /dev/null
					echo "✅ Get OK"

					echo "➡️ Updating TODO"
					curl -s -f -X PUT "$BASE_URL/todos/$TODO_ID" \
					  -H "Content-Type: application/json" \
					  -d '{"text":"Learn python and more","checked":true}'
					echo "✅ Update OK"

					echo "➡️ Deleting TODO"
					curl -s -f -X DELETE "$BASE_URL/todos/$TODO_ID"
					echo "✅ Delete OK"

				'''
			}
		} 
		
		stage('Promote') { 
			steps { 
				sh ''' 
					git checkout master 
					git merge origin/develop 
					git push origin master 
				''' 
			}
		} 
	}
}