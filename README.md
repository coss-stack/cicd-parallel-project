                         Developer
                             |
                             | git push
                             v
                         GitHub
                             |
                             | Webhook
                             v
                     Jenkins Master
                             |
              +--------------+--------------+
              |                             |
              v                             v
        Jenkins Agent 1               Jenkins Agent 2
        docker-agent                   scan-agent
              |                             |
              |       PARALLEL              |
              +--------+----------+---------+
                       |          |
                       v          v
                    Testing    Security
                       |
                       +----------+
                                  |
                                  v
                            Docker Build
                                  |
                                  v
                              AWS ECR
                                  |
                                  v
                              AWS EKS
                                  |
                          +-------+-------+
                          |               |
                          v               v
                       Pod 1           Pod 2
                          |               |
                          +-------+-------+
                                  |
                                  v
                           LoadBalancer
                                  |
                                  v
                              End User
