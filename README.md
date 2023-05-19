---
page_type: sample
languages:
- jmx
products:
- Azure Load Testing
- Azure OpenAI
- Others
name: Load Testing Azure OpenAI with JMeter
description: "Config samples to perform load testing scenarios against Azure OpenAI, frontend, backend and Vector Search/DB"
---

# Azure Load Testing to benchmark Azure OpenAI and other E2E services

This repo helps to benchmark latency and concurrency levels for some of the services involved in GPT completions scenarios. A typical e2e QnA / Semantic Search project for a Enterprise GPT includes Azure OpenAI (aka AOAI), App Service (AAS) and Cognitive Search (ACS), all moving pieces that could create bottlenecks and unnecessary latencies. At the same time prompt complexity, length, turn history and geographical distribution will surely affect the user experience. Using GPT-4 vs other faster models like ChatGPT-turbo will surely affect the final performance.

Internally this repo leverages [Apache JMeter](https://jmeter.apache.org/) as an open source load tool and [Azure Load Testing](https://azure.microsoft.com/en-us/products/load-testing/) (ALT) as a PaaS test orchestrator. ALT simplifies the configuration, deployment and hosting of the load test scenarios while allowing sofisticated deployment scenarios like Private networks, Secured Secrets (i.e. AOAI keys) and dynamic body ingestions (sending diversified promts to avoid possible cache replies). 

You should start your tests with a "user like" approach, hitting the final endpoint and analyzing the results. If a high latency or low concurrency is found you can then focus on isolated tests for each breakdown piece (i.e. Embeddings API, Vector Search, AppService Frontend+Backend, etc)

![image](https://github.com/ignaciofls/LoadTest-AOAI/assets/38979090/7e80bb12-7df4-453e-92ab-190b56100ccc)

## Step by step guide

Follow [this tutorial](https://learn.microsoft.com/en-us/azure/load-testing/how-to-create-and-run-load-test-with-jmeter-script) to spin up an Azure Load Testing resource and use the [configuration file](./jmeter/config.jmx) and [prompts CSV](./jmeter/prompts.csv) in your own subscription.

## Tailor your tests

### Simple config.jmx

The [sample configuration file](./jmeter/config.jmx) is a basic test where 300 requests are sent over a minute, this can easily be adjusted to your real scenarios with some tuning from Apache JMeter. A simplified way of building it is with their [UI tutorial](https://jmeter.apache.org/usermanual/build-web-test-plan.html).

### More advanced scenarios

Other complex scenarios might include a sustained load during minutes or even unauthorized avalanche calls (Placeholder)

### Generate your own test datasets

Because of the nature of ChatGPT and GPT in general we can expect varied and long conversations to be handled by the e2e system. Setting a fixed prompt will not simulate well the reality, hence we suggest to take advantage of a [prepopulated CSV](./jmeter/prompts.csv) with sample prompts that will be randomized into the test threads. Note that most of the sample rows have been self populated with GPT-4, once you move into Production you can use the logs from real usage to tailor the test dataset to your real needs in terms of length, complexity, etc.

For example, this system message helps to generate other chat history samples: 

<em>"You are an AI assistant that helps to create test datasets with a format compatible with GPT Chat session history. For example: "{"messages":[{"role":"system","content":"You are an AI assistant that helps people find information."},{"role":"user","content":"when did Michael Jordan win his first ring in the NBA?"},{"role":"assistant","content":"Michael Jordan won his first NBA championship ring on June 12, 1991, when the Chicago Bulls defeated the Los Angeles Lakers in the NBA Finals."},{"role":"user","content":"cool, for which team was Rodman playing for?"},{"role":"assistant","content":"During the 1990-1991 NBA season, when Michael Jordan won his first championship, Dennis Rodman was playing for the Detroit Pistons."},{"role":"user","content":"ah, they were good. Tons of friction between those two teams. When did Isiah Tomas retire?"},{"role":"assistant","content":"Isiah Thomas retired from professional basketball after the 1993-1994 NBA season."},{"role":"user","content":"thanks for correcting my typo, noticed you gently fixed it"},{"role":"assistant","content":"You're welcome! I'm here to help and provide accurate information. If you have any more questions or need assistance, feel free to ask."},{"role":"user","content":"when did Penny Hardaway join Orlando?"}]}"</em>

along a user message like: <em>"Generate 10 examples of GPT Chat session history about customers complaining for slow internet connection"</em>. In this case the first completion was: <em>"Example 1: {"messages":[{"role":"system","content":"You are an AI assistant that helps people with their internet connection issues."},
    {"role":"user","content":"My internet connection is really slow lately. What could be the problem?"},{"role":"assistant","content":"There could be several reasons for a slow internet connection, such as network congestion, router issues, or interference from nearby devices. First, try restarting your modem and router to see if that resolves the issue."},{"role":"user","content":"I already tried restarting my modem and router, but it didn't help."},
    {"role":"assistant","content":"In that case, you might want to check for any updates to your router's firmware, or try changing the Wi-Fi channel to reduce interference. If the issue persists, please contact your internet service provider for further assistance."}]}"</em>

## Secure your tests

### Key vault secrets
Following security best practice, you should not use your key in clear in the .jmx config file, instead you can use a variable and keep the actual key in Azure Key Vault. See more guidance in the [following link](https://learn.microsoft.com/en-us/azure/load-testing/how-to-parameterize-load-tests#secrets)

### VNET 
Most likely your final deployment will be hardened and behind private networks, the [following article](https://learn.microsoft.com/en-us/azure/load-testing/concept-azure-load-testing-vnet-injection) shows the different possibilities to inject traffic from the Azure Load Testing tool into your boundary GPT deployment. 

## Results and reports
### Out of the box results
The results are sketched out of the box containing valuable info (latency, HTTP code, etc) along the thread run. See the [documentation](https://learn.microsoft.com/en-us/azure/load-testing/how-to-troubleshoot-failing-test?tabs=portal) for more details and the jmx itself for a list of data to include 

![image](https://github.com/ignaciofls/LoadTest-AOAI/assets/38979090/dc48e3a8-14c8-4880-a1b1-35facaca202e)

### Compare different results
Different variables in the test will lead to different patterns in latency and max concurrency. Compare your results as seen [here](https://learn.microsoft.com/en-us/azure/load-testing/how-to-compare-multiple-test-runs)

### Customize the output csv
Quite often you not only need HTTP code, latency and timestamp but also the returned completion or other headers from the server. In that case you might want to check Listener and other config from JMeter as seen [here](https://www.blazemeter.com/blog/response-data-in-jmeter#:~:text=to%20a%20file.-,Saving%20Response%20Data%20with%20Listeners,-The%20first%20option)


