# Develop and Secure APIs with Apigee X: Challenge Lab

Quest: [#194 challenge lab](https://partner.cloudskillsboost.google/focuses/32743?parent=catalog)

## Solution for Task 1: Proxy the Cloud Translation API

### Step 1: Check enabled API and setup Service Account

On GCP Cloud Shell:

```sh
# Optional enable Translation API
gcloud services enable translate.googleapis.com

# Create service account `apigee-proxy` with role: Logging > Logs Writer
gcloud iam service-accounts create apigee-proxy --display-name="apigee-proxy"
gcloud projects add-iam-policy-binding ${GOOGLE_CLOUD_PROJECT} \
    --member="serviceAccount:apigee-proxy@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com" \
    --role="roles/logging.logWriter"
echo "Apigee deployer email: apigee-proxy@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com"
```

### Step 2: Create API Proxy and fix Authentication

1. Navigate to [Apigee UI](https://apigee.google.com)

2. Create a **Reverse Proxy** (don't click on the open api link)

| Field      | Value                                                    |
| ---------- | -------------------------------------------------------- |
| Name       | translate-v1                                             |
| Base path  | /translate/v1                                            |
| Target Url | https://translation.googleapis.com/language/translate/v2 |

3. Complete the rest of the form leaving defaults

4. Click on **Edit Proxy** and then go to the **Develop** tab.

5. **Authentication using a GoogleAccessToken** -> 

    a. Select the `Target Endpoints > default` section and replace the `HTTPTargetConnection` with the following:

    ```xml
    <HTTPTargetConnection>
        <Authentication>
            <GoogleAccessToken>
                <Scopes>
                    <Scope>https://www.googleapis.com/auth/cloud-translation</Scope>
                </Scopes>
            </GoogleAccessToken>
        </Authentication>
        <URL>https://translation.googleapis.com/language/translate/v2</URL>
    </HTTPTargetConnection>
    ```

    b. Click **Save**.

6. **Deploy to eval** using the Service Account created in Step 1

---

## Solution for Task 2: Change the API request and response

Using the files in this repo, perform the following steps in order using the Apigee UI:

1.  Create propertyset > `lanugage.properties` as below
```
output=es
caller=en

```
2. Create proxy endpoint `translate`, path `/` and verb `POST`
3. Create proxy endpoint `getLanguages`, path `/lanugages`

4. For `translate` proxy endpoint, create request and response policies
    1. create Request `AssignMessage` policy (recall from [the Securing APIs lab](https://partner.cloudskillsboost.google/focuses/32741?parent=catalog)) with name **AM-BuildTranslateRequest** and contents in [policies/AM-BuildTranslateRequest.xml file](policies/AM-BuildTranslateRequest.xml)
    2. create Response `AssignMessage` policy with name **AM-BuildTranslateResponse** and contents in [policies/AM-BuildTranslateResponse.xml file](policies/AM-BuildTranslateResponse.xml)
5. For `getLanguages` proxy endpoint, create request and response policies
    1. create Request `AssignMessage` policy with name **AM-BuildLanguageRequest** and contents in [policies/AM-BuildLanguageRequest.xml file](policies/AM-BuildLanguageRequest.xml)
    2. create Response `Javascript` policy with name **JS-BuildLanguageResponse** and javascript file name **buildLanguageResponse.js**
    3. Contents of **buildLanguageResponse.js** in [policies/buildLanguageResponse.js file](policies/buildLanguageResponse.js)

6. **Deploy to eval** using the Service Account created in Step 1

**IMPORTANT NOTE** It is of paramount importance that the javascript code uses double quotes (not single quotes), as the lab wouldn't pass single quoted code

---

## Solution for Task 3: Add API key verification and quota enforcement

1. create API product `translate-product`, with operation allowing access to /, and methods GET & POST and quota'ed at 10 requests every 1 minute
2. create dev with email `joe@example.com`
3. create app `translate-app`

4. To Proxy endpoints, preflow requests section, add VerifyAPIKey policy named `VAK-VerifyKey` replace contents from [policies/VAK-VerifyKey.xml file](policies/VAK-VerifyKey.xml)

5. To Proxy endpoints, preflow requests section, add Quota Policy named `Q-EnforceQuota` replace contents from [policies/Q-EnforceQuota.xml](policies/Q-EnforceQuota.xml)

6. **Deploy to eval** using the Service Account created in Step 1

---

## Solution for Task 4: Add message logging

On the default Proxy Endpoint, in the `translate` Flow's response section, after the `AM-BuildTranslateResponse`, add Message Logging policy named `ML-LogTranslation` with contents [policies/ML-LogTranslation.xml](policies/ML-LogTranslation.xml)

**Deploy to eval** using the Service Account created in Step 1

---

## Solution for Task 5: Rewrite a backend error message

1. On the Policies section click `+`, to add AssignMessage policy named `AM-BuildErrorResponse` with contents [policies/AM-BuildErrorResponse.xml](policies/AM-BuildErrorResponse.xml)

2. On the default Target Endpoint, add a `FaultRules` section after `</HTTPTargetConnection>` as follows:

    ```xml
<TargetEndpoint name="default">
    <FaultRules>
        <FaultRule name="400">
            <Step>
                <Name>AM-BuildErrorResponse</Name>
            </Step>
            <Condition>(response.status.code == 400) and (fault.name == "ErrorResponseCode")</Condition>
        </FaultRule>
    </FaultRules>
    ```

3. **Deploy to eval** using the Service Account created in Step 1

