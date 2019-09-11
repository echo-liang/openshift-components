---

title: "OWASP ZAP Security Vulnerability Scanning"
description: "The OWASP Zed Attack Proxy (ZAP) automatically finds security vulnerabilities in web applications."

---


# OWASP ZAP Security Vulnerability Scanning

The OWASP Zed Attack Proxy (ZAP) automatically finds security vulnerabilities in web applications.

The tool runs in the pipeline with several pre-packaged options:

* zap-api-scan.py - [For more details](https://github.com/zaproxy/zaproxy/wiki/ZAP-API-Scan)
* zap-baseline.py - [For more details](https://github.com/zaproxy/zaproxy/wiki/ZAP-Baseline-Scan)
* zap-full-scan.py - [For more details](https://github.com/zaproxy/zaproxy/wiki/ZAP-Full-Scan)

Please see the [original repository](https://github.com/rht-labs/owasp-zap-openshift) for more details on how this image was built.

Common options for the baseline scan are:

**Usage:** zap-baseline.py -t <target> [options]
    
    -t target         target URL including the protocol, eg https://www.example.com

**Options:**
    
    -c config_file    config file to use to INFO, IGNORE or FAIL warnings
    -u config_url     URL of config file to use to INFO, IGNORE or FAIL warnings
    -g gen_file       generate default config file (all rules set to WARN)
    -m mins           the number of minutes to spider for (default 1)
    -r report_html    file to write the full ZAP HTML report')
    -w report_md      file to write the full ZAP Wiki (Markdown) report
    -x report_xml     file to write the full ZAP XML report
    -a                include the alpha passive scan rules as well
    -d                show debug messages
    -P                specify listen port
    -D                delay in seconds to wait for passive scanning
    -i                default rules not in the config file to INFO
    -j                use the Ajax spider in addition to the traditional one
    -l level          minimum level to show: PASS, IGNORE, INFO, WARN or FAIL, use with -s to hide example URLs
    -n context_file   context file which will be loaded prior to spidering the target
    -p progress_file  progress file which specifies issues that are being addressed
    -s                short output format - dont show PASSes or example URLs
    -z zap_options    ZAP command line options e.g. -z "-config aaa=bbb -config ccc=ddd"
      
### Integrating OWASP ZAP Scanning and Reporting into your Project

To make the results of your ZAP security vulnerability scanning more accessible you can integrate the scan results into a SonarQube project report.  Detailed documentation and examples can be found in the [SonarQube on OpenShift](https://github.com/BCDevOps/sonarqube) project, which leverages the `openshift/jenkins-slave-zap` image generated from this project's source.

### Experiment with running OWASP ZAP in a pipeline

The simplest way to experiment with running ZAP in a pipeline is to include the following code in your pipeline Jenkinsfile:

```
podTemplate(label: 'owasp-zap', name: 'owasp-zap', serviceAccount: 'jenkins', cloud: 'openshift', containers: [
  containerTemplate(
    name: 'jnlp',
    image: '172.50.0.2:5000/openshift/jenkins-slave-zap',
    resourceRequestCpu: '500m',
    resourceLimitCpu: '1000m',
    resourceRequestMemory: '3Gi',
    resourceLimitMemory: '4Gi',
    workingDir: '/home/jenkins',
    command: '',
    args: '${computer.jnlpmac} ${computer.name}'
  )
]) {
  stage('ZAP Security Scan') {
    node('owasp-zap') {
        //the checkout is mandatory
        echo "checking out source"
        echo "Build: ${BUILD_ID}"
        checkout scm
        dir('zap') {
            def retVal = sh returnStatus: true, script: '/zap/zap-baseline.py -r index.html -t <your url>'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: '/zap/wrk', reportFiles: 'index.html', reportName: 'ZAP Full Scan', reportTitles: 'ZAP Full Scan'])
            echo "Return value is: ${retVal}"
            }
    }
  }
}
```