# Single Node Patterns

The goal of containers is to establish boundaries around specific resources eg. CPU and RAM. On a higer level, this boundary delineates team ownership and separation of concerns.

An example is an application with 2 components: a user-facing application server and a background configuration file loader. The user-facing request latency is the highest priority as that signficantly impacts the users. On the other hand, the background configuration file loader when delayed even during high user-request volumes will be okay. It does not impact the quality of service of the user. Hence, this is a good example of separating the user-facing service and the background service into different containers. This way, they each can be assigned different resource requirements and priorities.
