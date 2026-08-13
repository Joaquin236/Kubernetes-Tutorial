## Quiz de JSON Path_1 --> https://mmumshad.github.io/json-path-quiz/index.html#!/?questions=questionskub1

- 1 --> The Full JSON is a Dictionary
- 2 --> The JSON have 4 objects
- 3 --> The apiVersion field is a string value
- 4 --> The metadata field is a dictionary object
- 5 --> The containers field is a list of dictionaries
- 6 --> kubectl get pods -o=jsonpath=kind --> ["Pod"]
- 7 --> kubectl get pods -o=jsonpath=metadata.name --> ["nginx-pod"]
- 8 --> kubectl get pods -o=jsonpatch=spec.nodeName --> ["node01"]
- 9 --> kubectl get pods -o=jsonpatch=$.spec.containers[0] --> [{"image": "nginx:alpine","name": "nginx"}]
- 10--> kubectl get pods -o=jsonpatch=$.spec.containers[0].image  --> ["nginx:alpine"]
- 11--> kubectl get pods -o=jsonpatch=$.status.phase --> ["Pending"]
- 12--> kubectl get pods -o=jsonpatch=$.status.containerStatuses.[1].state.waiting.reason --> ["ContainerCreating"]
- 13--> kubectl get pods -o=jsonpatch=$.status.containerStatuses.[1].restartCount --> [2]
- 14--> kubectl get pods -o=jsonpatch=$.status.containerStatuses.[1].restartCount --> [2]
