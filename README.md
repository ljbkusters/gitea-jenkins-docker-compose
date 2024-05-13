# Gitea/Jenkins home dev setup

This repo is a bread-crumb for setting up a Jenkins instance coupled to a Gitea server with dynamic docker pipeline worker nodes. Ideally I would like to automate all these steps but I don't know if I will find the time to do so.

1. Install docker environment using docker-compose

`docker-compose -f docker-compose.yaml up`

2. Visit localhost:3000 from a browser and setup Gitea (on host)
3. Visit localhost:8080 from a browser and setup Jenkins (on host)
4. Download the Gitea and Docker plugins for Jenkins
    1. Visit localhost:8080 (Jenkins server)
    2. Go to "Manage Jenkins"
    3. Select plugins -> available plugins
    4. Install Gitea and Docker (not Docker-Pipeline!) plugins
5. Add an Organization to Gitea
    1. Go to localhost:3000 (Gitea server)
    2. Click on "Organization" (next to "Repository")
    3. Click on the `+` symbol and add an organization
    4. Give it a meaningful name (like `MyCompany`)
6. Add allowed webhook hosts to gitea server
    1. Go to `gitea/gitea/conf/app.ini`
    2. Add the following
    ```ini
    [webhook]
    ALLOWED_HOST_LIST = *
    ```

    TODO: this wildcard is probably risky, would be nice to add a specific host

7. Add a test repository to the newly created organization
    1. On the Gitea server, go to the newly created organization (`MyCompany`)
    2. Add a new repository
    3. Push the files under `./jenkins-test/` to this repository
8. Setup Gitea plugin
    1. Visit localhost:8080 (Jenkins server)
    2. Go to "Manage Jenkins"
    3. Select "System"
    4. Scroll down to Jenkins Server and enter `http://jenkins-server:8080/` which
       will make sure that gitea uses this address in its webhooks, which is within the
       exposed docker compose network.

       TODO: altough this fixes the gitea webhook issue, this is bad in the sense that
       Jenkins will now refer to itself with this address when sending automated emails and similar things, even though we access it from an outside OS on http://localhost:8080
       (or some portproxy).
    5. Now scroll down to gitea
    5. Give it a meaningful name (like `gitea`)
    6. Enter the Gitea server address. It can be reached on `http://gitea-server:3000`.
       NOTE: you should see a confirmation that the server is found and a version number
       which should correspond to the version number of the gitea-server.
    7. Add user credentials. This can be the system admin or a dedicated Jenkins account
       on the Gitea server. This will be the account managing webhooks and looking for
       pushed changes.
9. Add a Gitea organization to Jenkins
    1. Visit localhost:8080 (Jenkins server)
    2. Click on "New Job"
    3. Add an organization folder, give it a meaningful name
    4. Under repository sources, add a Gitea Organization
    5. Select the same credentials provided earlier
    6. In the `Owner` section add the organization defined in step 5
       (i.e. `MyCompany`)
    7. Save the changes. You will be taken to a page where the organization
    is scanned for repositories with Jenkinsfiles. It should find the Jenkinsfile
    in the repository you added in step 6.
    8. The Jenkinsfile should run without problems, the build should succeed.