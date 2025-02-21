import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

import KubernetesExample from "/examples/k8s/http-user-agent-filter.mdx";
import AgentCliExample from "/examples/agent-cli/http-user-agent-filter.mdx";
import AgentConfigExample from "/examples/agent-config/http-user-agent-filter.mdx";
import SshExample from "/examples/ssh/http-user-agent-filter.mdx";
import GoSdkExample from "/examples/go-sdk/http-user-agent-filter.mdx";
import NodejsSdkExample from "/examples/nodejs-sdk/http-user-agent-filter.mdx";
import PythonSdkExample from "/examples/python-sdk/http-user-agent-filter.mdx";
import RustSdkExample from "/examples/rust-sdk/http-user-agent-filter.mdx";

# User Agent Filter

## Overview

The User Agent Filter module enables you to block bots, crawlers, or certain
browsers from accessing your web application. It allows or denies traffic based
on the `User-Agent` header of incoming HTTP requests.

The `User-Agent` header contains information
about the requesting browser or application, including its name, version, and
operating system.

You define a set of regular expression rules that either allow or deny HTTP
requests if they match the `User-Agent` header.

## Example Usage

<Tabs groupId="connectivity" queryString="cty">
	<TabItem value="agent-cli" label="Agent CLI" default>
		<AgentCliExample />
	</TabItem>
	<TabItem value="agent-config" label="Agent Config">
		<AgentConfigExample />
	</TabItem>
	<TabItem value="ssh" label="SSH">
		<SshExample />
	</TabItem>
	<TabItem value="go-sdk" label="Go">
		<GoSdkExample />
	</TabItem>
	<TabItem value="nodejs-sdk" label="NodeJS">
		<NodejsSdkExample />
	</TabItem>
	<TabItem value="python-sdk" label="Python">
		<PythonSdkExample />
	</TabItem>
	<TabItem value="rust-sdk" label="Rust">
		<RustSdkExample />
	</TabItem>
	<TabItem value="k8s" label="Kubernetes Controller">
		<KubernetesExample />
	</TabItem>
</Tabs>

## Behavior

### Rule Evaluation

The User Agent Filter module will check each HTTP request's `User-Agent` header
value against the list of defined allow and deny regular expression rules.

- Requests without the `User-Agent` header will be denied by default. To allow these requests, you can add an `allow` rule for empty strings: `^$`
- If a request has multiple `User-Agent` headers, only the first header will be checked.

### Match Behavior

Requests are allowed if they match any allow rule or fail to match all deny
rules.

You can override `deny` rules by defining an `allow` rule that is more
specific, for instance blocking all Google bots except Google Images:

```json
{
	"allow": ["(?i)google-images"]
	"deny": ["(?i)google", "[Bb]ing"]
}
```

## Reference

### Configuration

###### **Agent Configuration**

| Parameter                   | Description                                                                  |
| --------------------------- | ---------------------------------------------------------------------------- |
| **User Agent Filter Allow** | A set of regular expressions used to match User-Agents that will be allowed. |
| **User Agent Filter Deny**  | A set of regular expressions used to match User-Agents that will be denied.  |

### Upstream Headers {#upstream-headers}

This module does not add any upstream headers.

### Errors

| Code                                      | HTTP Status | Error                                                       |
| ----------------------------------------- | ----------- | ----------------------------------------------------------- |
| [ERR_NGROK_3211](/errors/err_ngrok_3211/) | `403`       | The server does not authorize requests from your user-agent |

### Events

When the User Agent Filter module is enabled, it populates the following
fields in the
[http_request_complete.v0](/obs/reference/#http-request-complete) event:

| Fields                       |
| ---------------------------- |
| `user_agent_filter.decision` |

### Pricing

This module is available on all plans.

## Try it out

Run ngrok with User Agent Filter's `allow` and `deny` set to the following:

```bash
ngrok http 80 \
  --domain your-domain.ngrok.app \
  --ua-filter-allow="(GoingMerry/(\d)+.(\d)+)","(GomuGomu/(\d)+.(\d)+)" \
  --ua-filter-deny="(Xebec/(\d)+.(\d)+)"
```

Then make requests to your ngrok domain with the following:

```bash
curl --location https://your-domain.ngrok.app -H 'Content-Type: text/plain' -A 'GoingMerry/1.1' --data 'https://www.youtube.com/watch?v=djyTG19Achg' -k -v
curl --location https://your-domain.ngrok.app -H 'Content-Type: text/plain' -A 'GomuGomu/1.1' --data 'https://www.youtube.com/watch?v=djyTG19Achg' -k -v
curl --location https://your-domain.ngrok.app -H 'Content-Type: text/plain' -A 'Xebec/1.1' --data 'https://www.youtube.com/watch?v=djyTG19Achg' -k -v
curl --location https://your-domain.ngrok.app -H 'Content-Type: text/plain' -A '' --data 'https://www.youtube.com/watch?v=djyTG19Achg' -k -v
curl --location https://your-domain.ngrok.app -H 'Content-Type: text/plain' -A 'TwitterBot/1.1' --data 'https://www.youtube.com/watch?v=djyTG19Achg' -k -v
```
