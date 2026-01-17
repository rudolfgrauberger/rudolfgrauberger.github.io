---
layout: post
comments: true
title:  "Docker für Windows: Connect to localhost:2375 [localhost/127.0.0.1, localhost/0:0:0:0:0:0:0:1] failed"
description: "Beim Benutzen des dockerfile-maven-plugin kann man unter Windows kein Image bauen."
date:   2019-04-27 20:15:00 +0100
category: Entwicklung
hideToc: true
tags: [docker, windows, dev, localhost, docker for windows, maven, maven-plugin, legacy, archiv]
---

Beim Benutzen des `dockerfile-maven-plugin` erhält man unter Windows beim Kompilieren die Fehlermeldung.

```log
[ERROR] Failed to execute goal com.spotify:dockerfile-maven-plugin:1.4.10:build (default) on project project-service: Could not build image: java.util.concurrent.ExecutionException: com.spotify.docker.client.shaded.javax.ws.rs.ProcessingException: com.spotify.docker.client.shaded.org.apache.http.conn.HttpHostConnectException: Connect to localhost:2375 [localhost/127.0.0.1, localhost/0:0:0:0:0:0:0:1] failed: Connection refused: connect -> [Help 1]
org.apache.maven.lifecycle.LifecycleExecutionException: Failed to execute goal com.spotify:dockerfile-maven-plugin:1.4.10:build (default) on project project-service: Could not build image
        at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:213)
        at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:154)
        ...
```

### Lösung/Workaround

<!--more-->

{{< archiv-note >}}

1. Auf das Docker-Icon im System-Tray mit der rechten Maustaste und da auf **Settings** klicken

    ![System-Tray > Docker Settings](../SystemTrayDockerSettings.png)

2. Unter **General** ein Häkchen bei `Expose daemon on tcp://localhost:2375 without TLS` setzen

    ![Docker Settings > General > Expose daemon...](../DockerSettingsGeneralExposeDaemon.png)

3. Kurz warten, bis Docker wieder läuft (wird links unten in den Settings angezeigt)

    ![Docker Settings > Docker is running](../DockerSettingsDockerIsRunning.png)

4. Jetzt erneut Kompilieren

***Hinweis:*** Wie man bereits den Hinweis in den Docker-Einstellungen entnehmen kann, macht dies einem angreifbar. Aus diesem Grund bitte die Option **wieder deaktivieren**, sobald man es nicht mehr benötigt!

---
Quellen

ramicon, DOCKER COMMUNITY FORUMS, [https://forums.docker.com/t/cannot-connect-to-the-docker-daemon-at-unix-var-run-docker-sock-is-the-docker-daemon-running/43371/5](https://forums.docker.com/t/cannot-connect-to-the-docker-daemon-at-unix-var-run-docker-sock-is-the-docker-daemon-running/43371/5), 27.04.2019