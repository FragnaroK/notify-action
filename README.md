NotifY (IFTTT Integration Notification Action)
=====================================

This GitHub Action allows you to send notifications to IFTTT based on different events in your repository. (using Webhooks)

Inputs
------

### `type` (required)

Notification type (Success, Error, Pending, Warning, Info, Debug)

### `repoName` (required)

Repository name

### `branch`

Branch name

### `details`

Notification message

### `repoLink`

Repository link

### `IFTTT_EVENT`

IFTTT event name (default: 'ga\_notify')

### `IFTTT_KEY` (required)

IFTTT key

### `HEADER`

IFTTT header (default: '{"Content-Type": "application/json"}')

Usage
-----
 

```yaml
name: Your Workflow  
on:   
  push:     
    branches:     
      - main 
jobs:   
  notify:     
    runs-on: ubuntu-latest     
    steps:     
      - name: Checkout repository       
        uses: actions/checkout@v4      
      - name: Notify IFTTT       
        uses: fragnarok/notify-action@v2
        with:         
          type: 'error'         
          repoName: 'exampleRepo'         
          branch: 'main'         
          details: 'Workflow failed!'         
          repoLink: 'https://github.com/user/repo'        
          IFTTT_KEY: ${{ secrets.IFTTT_KEY }}
```

The Object sent to IFTTT will look like this:

```json
{
  "status": "error",
  "repoName": "YourRepoName",
  "repoBranch": "main",
  "repoLink": "https://github.com/<your-username>/<repo-name>",
  "details": "Workflow failed!",
}
```


A possible way to handle this data in IFTTT could be:

> Filter code once the event is received


```typescript

type GAStatus = 'error' | 'success' | 'warning' | 'pending' | 'debug' | 'info';

interface GANotification {
  icon: string;
  repoName: string;
  status?: GAStatus;
  repoBranch?: string;
  repoLink?: string;
  details?: string;
}

function getIcon(status: GAStatus) {
  switch (status.toLocaleLowerCase()) {
    case 'error':
      return '❌';
    case 'success':
      return '✅';
    case 'warning':
      return '⚠️';
    case 'pending':
      return '⏱️';
    case 'debug':
      return '🔨';
    case 'info':
      return '❕';
    default:
      return '🤖';
  }
}

function parsePayload(payload: string) {
  const parsedJSON = JSON.parse(payload);

  const parsedNotification: GANotification = {
    icon: getIcon(parsedJSON.status ? parsedJSON.status : 'info'),
    status: (parsedJSON.status ? parsedJSON.status : 'info').toLocaleUpperCase(),
    repoName: parsedJSON.repoName.toLocaleUpperCase(),
    repoBranch: parsedJSON.repoBranch,
    repoLink: parsedJSON.repoLink,
    details: parsedJSON.details,
  };

  return parsedNotification;
}

function main() {
  const {
    icon,
    status,
    repoName,
    repoBranch,
    repoLink,
    details
  } = parsePayload(MakerWebhooks.jsonEvent.JsonPayload);

  Telegram.sendMessage.setText(`
<b>${icon} ${status} - ${repoName} </b>(<a href="${repoLink}">${repoBranch}</a>
)${details}
  `)
}

main();
```

Feel free to customize the workflow to fit your specific use case.
