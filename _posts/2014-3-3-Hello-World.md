---
layout: post
title: You're up and running!
published: true
---
## Background


I am working on developing a rules engine designed to handle ISO20022 standard payment (SWIFT) and securities messages. It handles parsing, validation and enrichment of message and provides business users access to these features via a simple English like DSL. More about this later.

I had been deploying to virtual machines on the cloud, but wanted a solution that was more cost effective and scalable. Cloud Run is a service by Google Cloud Platform to run your stateless HTTP containers without worrying about provisioning machines, clusters or autoscaling. 

I will talk about how to deploy your microservice to Google Cloud using a CI/CD pipeline. My app is a Scala app built using SBT, but you can use the same pipeline for apps build using other platforms too. You start off by building a Docker image (this post) and then deploy the container on GCP Vloud run.

Dockerizing a Scala application is pretty easy. 

## Add the SBT native packager to project/plugins.sbt. This lets you build application packages in native formats
	addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.7.5")

## Package your application and create launch scripts using the [Java app packaging plugin](https://sbt-native-packager.readthedocs.io/en/latest/archetypes/java_app/index.html#java-app-plugin)

	enablePlugins(JavaAppPackaging)
    
## Ask SBT to build Docker images. You will need a Docker client has to be installed
 
 	enablePlugins(DockerPlugin)
    
## Check it out

    sbt docker:publishLocal
    
    docker run --rm -p8080:8080 payment-rules:1.0
 
## This should work out of the box, but you can customise each steps

Customise the packaging and launch scripts (e.g. setting environment variables or JVM parameter using the various JavaAppPackaging plugin settings)

	mainClass in (Compile, run) := Some("payrules.server.PayRuleServerMain")
    bashScriptExtraDefines += """addApp "-http.port=:${PORT:-8080}""""

Customise [Docker image generation](https://www.scala-sbt.org/sbt-native-packager/formats/docker.html)

    packageName in Docker := "payment-rules/api"
    dockerRepository := Some("eu.gcr.io")


## Optimise Docker image

By default the sbt docker plugin uses openjdk:latest as the base image. You can customise this further

	dockerBaseImage := "openjdk:jre-alpine"

If you want to move to a lightweight OS like Alpine or BusyBox, add the [following plugin](https://www.scala-sbt.org/sbt-native-packager/latest/api/com/typesafe/sbt/packager/archetypes/scripts/AshScriptPlugin$.html) to generate Ash compliant scripts.

	enablePlugins(AshScriptPlugin)


