<p align="center">
  <a href="http://nestjs.com/" target="blank">
    <img src="https://nestjs.com/img/logo-small.svg" width="120" alt="Nest Logo" />
  </a>
  <a href="https://kafka.js.org">
    <img src="https://raw.githubusercontent.com/tulios/kafkajs/master/logo/v2/kafkajs_circle.svg" alt="KafkaJS Logo" width="125" height="125">
  </a>
</p>

# NestJS + KafkaJS

Integration of KafkaJS with NestJS to build event driven microservices.


## Setup

Add the KafkaModule with the settings:

```javascript
@Module({
  imports: [
    KafkaModule.register([
      {
        name: 'HERO_SERVICE',
        options: {
          client: {
            clientId: 'hero',
            brokers: ['localhost:9092'],
          },
          consumer: {
            groupId: 'hero-consumer'
          }
        }
      },
    ]),
  ]
  ...
})

```

Full settings can be found:

| Config | Options |
| ------ | ------- | 
| client       | https://kafka.js.org/docs/configuration | 
| consumer     | https://kafka.js.org/docs/consuming#options |
| producer     | https://kafka.js.org/docs/producing#options |
| serializer   | |
| deserializer | |
| consumeFromBeginning | true/false |
| | |



### Subscribing

Subscribing to a topic to accept messages.

```javascript
export class Consumer {
  constructor(
    @Inject('HERO_SERVICE') private client: KafkaService
  ) {}

  onModuleInit(): void {
    this.client.subscribeToResponseOf('hero.kill.dragon', this)
  }

  @SubscribeTo('hero.kill.dragon')
  async getWorld(@Payload() data: KafkaMessage): Promise<void> {
    ...
  }

}

```

### Producing

Send messages back to kafka.

```javascript
const TOPIC_NAME = 'hero.kill.dragon';

export class Producer {
  constructor(
    @Inject('HERO_SERVICE') private client: KafkaService
  ) {}

  async post(message: string = 'Hello world'): Promise<RecordMetadata[]> {
    const result = await this.client.send({
      topic: TOPIC_NAME,
      messages: [
        {
          key: '1',
          value: message
        }
      ]
    });

    return result;
  }

}

```

### Schema Registry support.

By default messages are converted to JSON objects were possible. If you're using
AVRO you can add the `SchemaRegistry` module to parse the messages.

In your `module.ts`:

```javascript

@Module({
  imports: [
    KafkaModule.register([
      {
        name: 'HERO_SERVICE',
        options: {
          client: {
            clientId: 'hero',
            brokers: ['localhost:9092'],
          },
          consumer: {
            groupId: 'hero-consumer'
          }
        },
        deserializer: new KafkaAvroResponseDeserializer()
      },
    ]),
    SchemaRegistry.register({
      url: 'http://localhost:8081'
    }),
  ]
  ...
})
```

In your `controller.ts`:

```javascript
export class Consumer {
  constructor(
    @Inject('HERO_SERVICE') private client: KafkaService
  ) {}

  onModuleInit(): void {
    this.client.subscribeToResponseOf('hero.kill.dragon', this)
  }

  @SubscribeTo('hero.kill.dragon')
  @AvroSchema('hero.kill.dragon', 1)
  async getWorld(@Payload() data: KafkaMessage): Promise<void> {
    ...
  }

}

```

The `AvroSchema` decorator accepts a version number or undefined, this is to either pull an exact version from the schema registry, or always
get the latest schema.

## TODO

* Tests
* Add Avro schema to outgoing messages.
* Get avro schema for key, as we are only getting the value.