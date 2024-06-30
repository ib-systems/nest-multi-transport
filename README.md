# nest-multi-transport
This module will allow you to pre-configure topics (initially tested for Kafka, but other transports should work)

```ts
@Controller()
export class KafkaController {
  /**
  * @param {string} stands for Global topics map key. handles <string|string[]> as value
  */
  @KafkaTopics('DYNAMIC_TOPICS_KEY') 
  async process(@Payload() data, @Ctx() context: KafkaContext) {
    console.log("Process message with dynamically defined topics.");
  }
}
```

#How it works
1. Specific "controller" decorator is created to add kafka topic candidate metadata.
2. Once `await app.init()` is called nestjs initialize all modules, providers and so on
3. OnModuleInit lifecycle hook runs and adds topics to map
```ts
@Injectable()
export class SomeService implements OnModuleInit {
  constructor(
    @Inject("KAFKA_TOPICS_MAP") private readonly kafkaTopicsMap: Map<string|symbol, string|string[]>
  ){ }

   async onModuleInit() {
    // You could fetch it from database or external API. Next lines are just for exampple
    this.kafkaTopicsMap.set("DYNAMIC_TOPICS_KEY", [
      'some_topic_1',
      'some_topic_2',
    ])
  }
```
4. Then KafkaDecoratorProcessorService processes all controller methods that decorated with specific decorator (ref 1st step) and decorates it with vanilla nestjs `EventPattern`.
```ts
Reflect.decorate(
  [EventPattern(topics)],
  type.prototype,
  prop,
  Reflect.getOwnPropertyDescriptor(type.prototype, prop),
);
```
# Initial implementation roadmap
- [ ] define module (root or feature?)
- [ ] define topics map
- [ ] process topics on specific controllers
