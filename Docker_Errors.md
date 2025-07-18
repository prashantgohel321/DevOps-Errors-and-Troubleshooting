# Docker Errors

<details>
<summary><strong style="color:red"> 01. Cannot connect to the Docker daemon </strong></summary>
<br>

### Cause
> Docker service is not running or user lacks permissions. 

### Solution
1. Start the Docker service: `sudo systemctl start docker`. 
2. Add the user to the Docker group: `sudo usermod -aG docker $USER`. 
3. `newgrp docker`.


</details>

<br>

<details>
<summary><strong style="color:red"> 02. Port is already in use </strong></summary>
<br>

### Cause
> Another process is bound to the same port.

### Solution
1. Stop the conflicting container: `docker stop <container-id>`. 
2. Change the container's port mapping. 


</details>

<br>

<details>
<summary><strong style="color:red"> 03. No space left on device </strong></summary>
<br>

### Cause
> Disk space is exhausted by Docker images and containers. 

### Solution
1. Remove unused resources: `docker system prune -a`. 


</details>

<br>

<details>
<summary><strong style="color:red"> 04. ImagePullBackOff" in Kubernetes </strong></summary>
<br>

### Cause
> Invalid image tag or registry issues. 

### Solution
1. Verify the image name and registry credentials.


</details>

<br>

<details>
<summary><strong style="color:red"> 05. Permission denied on bind mount </strong></summary>
<br>

### Cause
> Host directory permissions are restrictive.

### Solution
1. Update directory permissions: `chmod 777 <directory>`. 


</details>
