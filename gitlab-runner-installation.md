# Gitlab-Runner Installation
Recently, due to the team demand to replace Gerrit, I used docker to build a self-host GitLab and GitLab Runner for the company. One of the points mentioned here is GitLab Runner is a plug-in to help GitLab run CI, a bit like Jenkins Slave Node, used to cascade GitLab CI.

This is mainly divided into three parts

1.  Build GitLab
2.  Build and connect GitLab Runner
3.  Set up auto backup and restore

## Start Gitlab Runner

    docker run -d --name gitlab-runner 
    --restart always \ 
    -v var/run/docker.sock:/var/run/docker.sock \  
    -v /srv/gitlab-runner/config:/etc/gitlab-runner:Z \   
    gitlab/gitlab-runner:latest



## Find Registration Code
you could go to GitLab website to find Runner Registration Token which locate at **Admin Area** > **Overview** > **Runners**


## Register Runner

Replace `{runner-name}` to your container name.

    docker exec -it {runner-name} gitlab-runner register -n 
    --url http://{gitlab-ip}:9090 
    --registration-token {registration-token} 
    --clone-url http://{gitlab-ip}:9090 
    --executor docker --docker-image "docker:latest" 
    --docker-privileged
