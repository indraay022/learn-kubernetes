```
controlplane ~ ➜  kubectl get pod
NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          6m31s
webapp-2   2/2     Running   0          3m21s
```
```
controlplane ~ ➜  kubectl logs webapp-1
[2023-06-25 04:33:48,828] INFO in event-simulator: USER1 is viewing page3
[2023-06-25 04:33:49,830] INFO in event-simulator: USER4 logged out
[2023-06-25 04:33:50,830] INFO in event-simulator: USER2 logged out
[2023-06-25 04:33:51,831] INFO in event-simulator: USER2 logged out
[2023-06-25 04:33:52,832] INFO in event-simulator: USER3 logged out
[2023-06-25 04:33:53,834] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
```

```
controlplane ~ ➜ kubectl logs webapp-2 -c simple-webapp
[2023-06-25 04:36:55,395] INFO in event-simulator: USER2 logged in
[2023-06-25 04:36:56,396] INFO in event-simulator: USER4 is viewing page1
[2023-06-25 04:36:57,397] INFO in event-simulator: USER4 logged out
[2023-06-25 04:36:58,398] INFO in event-simulator: USER1 is viewing page3
[2023-06-25 04:36:59,399] INFO in event-simulator: USER4 logged out
[2023-06-25 04:37:00,400] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2023-06-25 04:37:00,400] INFO in event-simulator: USER1 is viewing page2
[2023-06-25 04:37:01,401] INFO in event-simulator: USER3 is viewing page1
[2023-06-25 04:37:02,403] INFO in event-simulator: USER4 is viewing page3
[2023-06-25 04:37:03,404] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
```