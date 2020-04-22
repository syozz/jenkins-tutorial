@Library('keptn-library')_
def keptn = new sh.keptn.Keptn()

node {
    properties([
        parameters([
         string(defaultValue: 'perfproject', description: 'Name of your Keptn Project for Performance as a Self-Service', name: 'Project', trim: false), 
         string(defaultValue: 'perfstage', description: 'Stage in your Keptn project used for Performance Feedback', name: 'Stage', trim: false), 
         string(defaultValue: 'serviceundertest', description: 'Servicename used to keep SLIs, SLOs, test files ...', name: 'Service', trim: false),
         choice(defaultValue: 'performance', choices: ['performance', 'performance_10', 'performance_50', 'performance_100', 'performance_long'], description: 'Test Strategy aka Workload, e.g: performance, performance_10, performance_50, performance_100, performance_long', name: 'TestStrategy', trim: false),
         choice(defaultValue: 'basic', choices: ['basic', 'perftest'], description: 'Decide which set of SLIs you want to evaluate. The sample comes with: basic and perftest', name: 'SLI'),
         string(defaultValue: 'http://simplenode.simpleproject-staging.keptn06-agrabner.demo.keptn.sh/', description: 'URI of the application you want to run a test against', name: 'DeploymentURI', trim: false),
         string(defaultValue: '60', description: 'How many minutes to wait until Keptn is done? 0 to not wait', name: 'WaitForResult'),
        ])
    ])

    stage('Initialize Keptn') {
        // keptn.downloadFile('https://raw.githubusercontent.com/keptn-sandbox/performance-testing-as-selfservice-tutorial/master/shipyard.yaml', 'keptn/shipyard.yaml')
        keptn.downloadFile("https://raw.githubusercontent.com/keptn-sandbox/performance-testing-as-selfservice-tutorial/master/slo_${SLI}.yaml", 'keptn/slo.yaml')
        keptn.downloadFile("https://raw.githubusercontent.com/keptn-sandbox/performance-testing-as-selfservice-tutorial/master/dynatrace/sli_${SLI}.yaml", 'keptn/sli.yaml')
        keptn.downloadFile('https://raw.githubusercontent.com/keptn-sandbox/performance-testing-as-selfservice-tutorial/master/jmeter/load.jmx', 'keptn/jmeter/load.jmx')
        keptn.downloadFile('https://raw.githubusercontent.com/keptn-sandbox/performance-testing-as-selfservice-tutorial/master/jmeter/jmeter.conf.yaml', 'keptn/jmeter/jmeter.conf.yaml')
        archiveArtifacts artifacts:'keptn/**/*.*'

        // Initialize the Keptn Project
        keptn.keptnInit project:"${params.Project}", service:"${params.Service}", stage:"${params.Stage}" // , shipyard:'shipyard.yaml'

        // Upload all the files
        keptn.keptnAddResources('keptn/sli.yaml','dynatrace/sli.yaml')
        keptn.keptnAddResources('keptn/slo.yaml','slo.yaml')
        keptn.keptnAddResources('keptn/jmeter/load.jmx','jmeter/load.jmx')
        keptn.keptnAddResources('keptn/jmeter/jmeter.conf.yaml','jmeter/jmeter.conf.yaml')
    }
    stage('Trigger Performance Test') {
        echo "Performance as a Self-Service: Triggering Keptn to execute Tests against ${params.DeploymentURI}"

        // send deployment finished to trigger tests
        def keptnContext = keptn.sendDeploymentFinishedEvent testStrategy:"${params.TestStrategy}", deploymentURI:"${params.DeploymentURI}" 
        String keptn_bridge = env.KEPTN_BRIDGE
        echo "Open Keptns Bridge: ${keptn_bridge}/trace/${keptnContext}"
    }
    stage('Wait for Result') {
        waitTime = 0
        if(params.WaitForResult?.isInteger()) {
            waitTime = params.WaitForResult.toInteger()
        }

        if(waitTime > 0) {
            echo "Waiting until Keptn is done and returns the results"
            def result = keptn.waitForEvaluationDoneEvent setBuildResult:true, waitTime:waitTime
            echo "${result}"
        } else {
            echo "Not waiting for results. Please check the Keptns bridge for the details!"
        }
    }
}