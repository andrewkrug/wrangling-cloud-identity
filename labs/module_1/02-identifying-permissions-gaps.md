# Identify Permissions Gaps at Scale

Permissions gaps are a top topic in the Identity and Access community. 
The most dangerous part of a policy that has a permissions gap is that it works!  

**Definition**: A permissions gap is defined by a condition where a policy has far more access than is needed to perform a job. 

> Example: A serverless function only needs to upload a single object to S3 from invocation. The policy applied is the AWS Managed Policy for the job role "S3FullAccess" granting far more permissive access to the bucket.

> Example: An engineer has the ability to DeleteAccessKey but this should only be a function performed by a service desk. 

## The Process for Permissions Gap Analysis 

Permissions gap analysis is complex and the consequence of getting it wrong is that we could potentially break the application in production if we apply a least privilege policy. *Talk about giving security a bad reputation*.

## Permissions Gap Methodology 

1. Identify the principal we want to reduce permissions for
2. Determine what access is used. *This is the hard part*
3. Compare that to the intent for the application
4. Generate a new managed policy to deploy

## Lab Instructions

In this lab we'll be using a colab python notebook in order to analyze data from CloudTrail using a new feature in AWS called Cloudtrail Lake. Cloudtrail Lake is a rapid method to setup an ANSI SQL interface to Cloudtrail data stored in JSON in S3. Cloudtrail Lake queries are cost effective, simple, and easy to execute. 

We will also use snippets of python in the iPython notebook to generate our new least privilege policy which could then be used to update the project.

Cloudtrail Lake will also be used in order to validate that our deployment of the new version of the policy has not caused problems in production.

1. Check the IAM policy for the verbs that are allowed.  Note this can be tricky and we will learn to 
master this in a future lesson

2. Identify the permissions in use within a window of time. Generally 7 days is sufficient
```
SELECT count (*) as TotalEvents,
    eventsource,
    eventname,
    useridentity.arn,
    errorCode,
    errorMessage
FROM 0dbef245-0ec5-4b12-b705-c38b6cede39d
WHERE 
    eventTime > DATE_ADD('week', -1, CURRENT_TIMESTAMP)
    AND useridentity.arn like '%arn:aws:sts::654654581435:assumed-role/privesc-high-priv-service-role/i-06f4a02c0a6b0c789%'
GROUP BY eventsource,
    eventname,
    errorCode,
    errorMessage,
    useridentity.arn
ORDER BY eventsource,
    eventname
```

3. Identify if there are any missing permissions for the role by running the following query against CloudTrail Lake

```sql
SELECT count (*) as TotalEvents,
    eventsource,
    eventname,
    useridentity.arn,
    errorCode,
    errorMessage
FROM 0dbef245-0ec5-4b12-b705-c38b6cede39d
WHERE (
        errorcode like '%Denied%'
        or errorcode like '%Unauthorized%'
    )
    AND eventTime > DATE_ADD('week', -1, CURRENT_TIMESTAMP)
    AND useridentity.arn like 'arn:aws:sts::654654581435:assumed-role/privesc-high-priv-service-role/i-06f4a02c0a6b0c789'
GROUP BY eventsource,
    eventname,
    errorCode,
    errorMessage,
    useridentity.arn
ORDER BY eventsource,
    eventname
```

## Congratulations!  

You've successfully completed a manual permissions gap analysis exercise.
Time to go and fix that role now that you know what the permissions should be!