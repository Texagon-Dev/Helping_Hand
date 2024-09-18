### Rate Limiting a WebSocket connection
Create a guard like so:
```
@Injectable()
export class WsThrottlerGuard extends ThrottlerGuard {
  async handleRequest({
    context,
    limit,
    ttl,
    throttler,
  }: {
    context: ExecutionContext;
    limit: number; // limit of requests before connection should be rate limited
    ttl: number; // how long requests are considered to be of the same session
    throttler: ThrottlerOptions; 
  }): Promise<boolean> {
    const client = context.switchToWs().getClient();
    const ip = client.conn.remoteAddress;
    const key = this.generateKey(context, ip, throttler.name); 
    // Increment number of requests made by this key (which is unique for each user and session)
    const out = await this.storageService.increment(
      key,
      ttl,
      limit,
      60000, // blocked for a minute
      'default', // name of the throttler
    );
    if (out.totalHits > limit) {
      throw new ThrottlerException();
    }
    return true;
  }
}
```
This guard can then be applied to a websocket controller or event using UseGuards or to the whole module. Docs: https://docs.nestjs.com/security/rate-limiting