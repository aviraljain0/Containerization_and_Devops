# Hands-on Task: Run and Manage a "Hello Web App" (httpd)

## Objective
Deploy and manage a simple Apache-based web server and:
* **Verify** it is running
* **Modify** it
* **Scale** it
* **Debug** it

---

## 1. Deploying a Simple Pod
* **Run Pod:** `kubectl run apache-pod --image=httpd`
![](../Screeshots/Apache_server_s/aps1.png)
* **Check Status:** `kubectl get pods`
* **Inspect:** `kubectl describe pod apache-pod`
    * Focus on the image (`httpd`), port (`80`), and events.
    ![](apache_folder/2.png)
* **Access:** `kubectl port-forward pod/apache-pod 8081:80`
![](apache_folder/3.png)
    * Open `http://localhost:8081` to see "It works!".
    ![](apache_folder/4.png)
* **Cleanup:** `kubectl delete pod apache-pod`
![](apache_folder/5.png)
    * Note: Pods do not self-heal; they disappear permanently when deleted.

---

## 2. Transitioning to a Deployment
* **Create Deployment:** `kubectl create deployment apache --image=httpd`
![](apache_folder/6.png)
* **Expose App:** `kubectl expose deployment apache --port=80 --type=NodePort`
![](apache_folder/8.png)
* **Access via Service:** `kubectl port-forward service/apache 8082:80`
![](apache_folder/7.png)
---

## 3. Scaling
* **Scale Up:** `kubectl scale deployment apache --replicas=2`
    * This ensures multiple pods run the same application.
![](apache_folder/9.png)



---

## 4. Debugging and Self-Healing
* **Simulate Failure:** Set the image to a non-existent one:
  * `kubectl set image deployment/apache httpd=wrongimage`
![](apache_folder/10.png)
* **Diagnose:** Look for `ImagePullBackOff` in `kubectl describe pod`.
![](apache_folder/11.png)
* **Fix:** Revert to the correct image: `kubectl set image deployment/apache httpd=httpd`.
* **Observe Healing:** Delete one pod and watch Deployment recreate it automatically.
![](apache_folder/12.png)

---

## 5. Terminal Management (Port-Forwarding)
* **Why it blocks:** It creates a live foreground tunnel.
* **Backgrounding:** Use `&` to run in background.
* **Management:**
    * Identify: `jobs` or `ps aux | grep port-forward`
    * Stop: `kill %1` or `kill <PID>`
    * Recommended: Use `tmux` for persistent sessions.

---

## Final Cleanup
* `kubectl delete deployment apache`
* `kubectl delete service apache`
![](apache_folder/14.png)
