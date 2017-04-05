#!groovy

node {
    // Setup the Docker Registry (Docker Hub) + Credential 
    registry_url = "https://index.docker.io/v1/" // Docker Hub
    docker_creds_id = "shaposhnikoff-Dockerhub" // name of the Jenkins Credentials ID
    build_tag = "testing" // default tag to push for to the registry
    stage 'Checking out GitHub Repo'
    git url: 'https://github.com/shaposhnikoff/docker-django-nginx-slack-sphinx.git'
    stage 'Building Django Container for Docker Hub'
    docker.withRegistry("${registry_url}", "${docker_creds_id}") {
        maintainer_name = "jayjohnson"
        container_name = "django-slack-sphinx"
        docker_env_file = "testing.env"
//      def docker_env_values = readProperties file: "./${docker_env_file}"
//        docker_env_values = readProperties file: 'testing.env'



        // Assign variables based off the env file
        default_root_volume = "/opt"
        doc_source_dir = "/tmp"
        doc_output_dir = "/tmp"
        static_output_dir = "/tmp"
        media_dir = "/tmp"

        stage "Building"
        echo "Building Django with docker.build(${maintainer_name}/${container_name}:${build_tag})"
        container = docker.build("${maintainer_name}/${container_name}:${build_tag}", 'django')
        try {
            // Start Testing
            stage "Running Django container"
            // Run the container with the env file, mounted volumes and the ports:
            docker.image("${maintainer_name}/${container_name}:${build_tag}").withRun("--name=${container_name} --env-file ${docker_env_file} -e ENV_SERVER_MODE=DEV -v ${default_root_volume}:${default_root_volume} -v ${doc_source_dir}:${doc_source_dir} -v ${doc_output_dir}:${doc_output_dir} -v ${static_output_dir}:${static_output_dir} -v ${media_dir}:${media_dir} -p 82:80 -p 444:443")  { c ->
                waitUntil {
                    sh "docker exec -t ${container_name} netstat -apn | grep 80 | grep LISTEN | wc -l | tr -d '\n' > /tmp/wait_results"
                    wait_results = readFile '/tmp/wait_results'

                    echo "Wait Results(${wait_results})"
                    if ("${wait_results}" == "1")
                    {
                        echo "Django is listening on port 80"
                        sh "rm -f /tmp/wait_results"
                        return true
                    }
                    else
                    {
                        echo "Django is not listening on port 80 yet"
                        return false
                    }
                } // end of waitUntil
                // At this point Django is running
                echo "Docker Container is running"
                // this pipeline is using 3 tests 
                // by setting it to more than 3 you can test the error handling and see the pipeline Stage View error message
                MAX_TESTS = 3
                for (test_num = 0; test_num < MAX_TESTS; test_num++) {     
                    echo "Running Test(${test_num})"
                    expected_results = 0
                    if (test_num == 0 ) 
                    {
                        // Test we can download the home page from the running django docker container
                        sh "docker exec -t ${container_name} curl -s http://localhost/home/ | grep Welcome | wc -l | tr -d '\n' > /tmp/test_results" 
                        expected_results = 1
                    }
                    else if (test_num == 1)
                    {
                        // Test that port 80 is exposed
                        echo "Exposed Docker Ports:"
                        sh "docker inspect --format '{{ (.NetworkSettings.Ports) }}' ${container_name}"
                        sh "docker inspect --format '{{ (.NetworkSettings.Ports) }}' ${container_name} | grep map | grep '80/tcp:' | wc -l | tr -d '\n' > /tmp/test_results"
                        expected_results = 1
                    }
                    else if (test_num == 2)
                    {
                        // Test there's nothing established on the port since nginx is not running:
                        sh "docker exec -t ${container_name} netstat -apn | grep 80 | grep ESTABLISHED | wc -l | tr -d '\n' > /tmp/test_results"
                        expected_results = 0
                    }
                    else
                    {
                        err_msg = "Missing Test(${test_num})"
                        echo "ERROR: ${err_msg}"
                        currentBuild.result = 'FAILURE'
                        error "Failed to finish container testing with Message(${err_msg})"
                    }
                    // Now validate the results match the expected results
                    stage "Test(${test_num}) - Validate Results"
                    test_results = readFile '/tmp/test_results'
                    echo "Test(${test_num}) Results($test_results) == Expected(${expected_results})"
                    sh "if [ \"${test_results}\" != \"${expected_results}\" ]; then echo \" --------------------- Test(${test_num}) Failed--------------------\"; echo \" - Test(${test_num}) Failed\"; echo \" - Test(${test_num}) Failed\";exit 1; else echo \" - Test(${test_num}) Passed\"; exit 0; fi"
                    echo "Done Running Test(${test_num})"
                    // cleanup after the test run
                    sh "rm -f /tmp/test_results"
                    currentBuild.result = 'SUCCESS'
                }
            }
        } catch (Exception err) {
            err_msg = "Test had Exception(${err})"
            currentBuild.result = 'FAILURE'
            error "FAILED - Stopping build for Error(${err_msg})"
        }
        stage "Pushing"
        container.push()
        currentBuild.result = 'SUCCESS'
    }
    stage 'Building nginx Container for Docker Hub'
    docker.withRegistry("${registry_url}", "${docker_creds_id}") {
        // Set up the container to build
        maintainer_name = "jayjohnson"
        container_name = "django-nginx"
        stage "Building Container"
        echo "Building nginx with docker.build(${maintainer_name}/${container_name}:${build_tag})"
        container = docker.build("${maintainer_name}/${container_name}:${build_tag}", 'nginx')
        // add more tests
        stage "Pushing"
        container.push()
        currentBuild.result = 'SUCCESS'
    }
    currentBuild.result = 'SUCCESS'
}
