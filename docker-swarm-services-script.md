# Docker Swarm - Managing Services

Services are configurations of tasks that are intended to be run on swarm nodes. Usually, services belong to a stack which is defined in a docker compose file. However, services can be created and managed using the command line interface.

## List services in a swarm

The ls commands lists all running services in a swarm.

```
$ docker service ls
```

## Inspect a service

The inspect command displays detailed information about a service. The output is displayed in JSON format.

```
$ docker service inspect [options] <service-name>
$ docker service inspect vote_vote
```

Options:  
-f : Formats the output using the given Go template  
--pretty : Prints the information in a human friendly format

To display the service information in pretty format (Indented format):

```
$ docker service inspect --pretty vote_vote
```

Use the -f option to display a specific information about a service.

```
$ docker service inspect -f '{{.Endpoint.Ports}}' vote_vote
```

## Show logs of a service

The logs command fetches the logs of a service or a task. The logs are messages send by the service/task to the standard output and standard error streams. The service logs include all logs of the service tasks.

```
$ docker service logs [options] <service-name> | <task_id>
$ docker service logs vote_db
$ docker service logs 332dm0jcsssg
```

Options:  
--follow : Continues streaming the new output from the service’s stdout and stderr streams.  
--details: Shows more log details  
--tail : Shows a specific number of lines from the end of the logs  
--since: Shows the service logs generated after a given date.

## List tasks of a service

The ps command lists the tasks, or replicas, that belong to a service.

```
$ docker service ps [options] <service-name>
$ docker service ps vote_redis
```

When a service is part of a stack, the service name has the format: \<stack-name>\_\<service-name-in-compose-file>

## Create a service

A service can be created using the command line interface instead of using a docker compose file. The command "service create" creates a new service based on the provided options.

```
$ docker service create [options] <image-name> [command][arguments]
```

The create command has multiple options that are equivalent to the options in the docker compose file.

Before creating a service, you may clean your swarm from all stacks by removing them using the "stack rm" command.

```
$ docker stack rm $(docker stack ls --format '{{.Name}}')
$ docker stack ls
```

The option --name set a name to the new service. The option -p publishes a port to the outside of the swarm.

```
$ docker service create --name web -p 8080:80 nginx:stable
$ docker service ls
```

Options:  
-d : Returns the prompt to user immediately  
--entrypoint: Overwrites the entrypoint option of an image  
-e : Sets environment variables for replicas  
--limit-cpu : Limits the number of cpu dedicated to the service  
--limit-memory : Limits the memory size dedicated to the service  
--mode : Can be either replicated or global (one replica per node)  
--network : Attaches the service to a network  
--mount : Attaches a filesystem to a service  
-p : Publishes a port  
--replicas : Sets the number of replicas  
--restart-condition : Restarts when condition is met (none | on-failure | any ) (default “any”)  
--delay-update : Sets the delay between updates  
--update-parallelism : Sets the number of tasks updated simultaneously

## Update a service

The docker swarms allows a rolling update of services. A rolling update is an update of a certain number of replicas of a service at the same time. This avoids downtime and makes the service available during service update.

```
$ docker service update [options] <service-name>
```

The options of the "service create" command can be used with the "service update" command.

```
$ docker service create --name web --replicas 5 -p 8080:80 nginx:latest
$ docker service update --publish-add 8181:80 web
```

The number of tasks, or replicas, to be updated at the same time is specified using the --update-parallelism option.

```
$ docker service update --update-parallelism 2 --image nginx:stable web
```

A time delay between updates of the service replicas can be set as follows:

```
$ docker service update --update-parallelism 2 --update-delay 5s --image nginx:mainline web
```

When an error occur during the update, a rollback to previous version of the service configuration can be performed.

```
$ docker service rollback <service-name>
$ docker service rollback web
```

## Scale a service

The scale command permits the increase or the decrease of the number of replicas of a service. The scaling of services may take some time to complete.

```
$ docker service scale <service-name>=<number-of-replicas> [options]
$ docker service scale vote_vote=3
```

To verify the number of replicas for the vote_vote service:

```
$ docker service ls
```

The REPLICAS column shows both the actual and desired number of tasks for the service.

The option -d returns the prompt to the user instead of waiting for the verification of scale to be stable.

```
$ docker service scale vote_vote=5 -d
```

## Remove a service

The rm command removes a service from a swarm. The command does not ask for confirmation to remove a service.

```
docker service rm <service-name>
docker service rm web
```
