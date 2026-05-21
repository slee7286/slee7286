ngrok http 5175    
ngrok : The term 'ngrok' is not recognized as the name of a cmdlet, function, 
script file, or operable program. Check the spelling of the name, or if a      
path was included, verify that the path is correct and try again.
At line:1 char:1
+ ngrok http 5175
+ ~~~~~
    + CategoryInfo          : ObjectNotFound: (ngrok:String) [], CommandNotFo  
   undException
    + FullyQualifiedErrorId : CommandNotFoundException

ngrok isn’t installed (or not on your PATH) on this machine.

Run this in PowerShell:

winget install --id ngrok.ngrok -e
Then close and reopen PowerShell and verify:

ngrok version
If you haven’t authenticated ngrok on this device yet:

ngrok config add-authtoken <YOUR_NGROK_AUTHTOKEN>
Then start your tunnel:

ngrok http 5175
If it still says not recognized, run:

where.exe ngrok
and I’ll help you fix PATH from that output.



winget install --id ngrok.ngrok -e
>> 
No package found matching input criteria.

I’ll check the exact winget package name on your machine and give you the fastest install path that works.

winget package exists, but the exact ID/source matters.

Use this command:

winget install --id Ngrok.Ngrok -e --source winget
I verified it installs successfully (ngrok version 3.3.1).

Then close and reopen PowerShell (PATH was updated during install), and run:

ngrok version
ngrok config add-authtoken <YOUR_TOKEN>
ngrok http 5175
If ngrok is still not recognized in the same session, run it directly once from:

& "$env:LOCALAPPDATA\Microsoft\WinGet\Packages\Ngrok.Ngrok_Microsoft.Winget.Source_8wekyb3d8bbwe\ngrok.exe" http 5175


winget install --id Ngrok.Ngrok -e --source winget
>>
Found an existing package already installed. Trying to upgrade the installed package...
No available upgrade found.
No newer package versions are available from the configured sources.

ngrok version
ngrok : The term 'ngrok' is not recognized as the name of a 
cmdlet, function, script file, or operable program. Check the     
spelling of the name, or if a path was included, verify that the  
path is correct and try again.
At line:1 char:1
+ ngrok version
+ ~~~~~
    + CategoryInfo          : ObjectNotFound: (ngrok:String) [],  
    CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException

ngrok is installed, but your shell still can’t resolve it via PATH/alias. I’ll inspect where the binary is and patch your user PATH so ngrok works directly in PowerShell.

I found the issue: ngrok.exe exists, but its install directory is not on your PATH. I’m adding that directory to your user PATH and current shell now, then verifying ngrok version.

Fixed: ngrok is installed and works now.

I verified:

ngrok version
# ngrok version 3.3.1
Your previous PowerShell session likely had stale PATH state. Use one of these:

Open a new PowerShell window, then run:
ngrok version
ngrok config add-authtoken <YOUR_TOKEN>
ngrok http 5175
Or run immediately by full path:
& "$env:LOCALAPPDATA\Microsoft\WinGet\Packages\Ngrok.Ngrok_Microsoft.Winget.Source_8wekyb3d8bbwe\ngrok.exe" http 5175


NAME:
  http - start an HTTP tunnel

USAGE:
  ngrok http [address:port | port] [flags]

DESCRIPTION:
  Starts a tunnel listening for HTTP/HTTPS traffic with a specific hostname.
  The HTTP Host header on incoming public requests is inspected to
  determine which tunnel it matches.

  HTTPS endpoints terminate TLS traffic at the ngrok server using the
  appropriate certificates. The decrypted, HTTP traffic is then forwarded
  through the secure tunnel and then to your local server. If you don't want
  your TLS traffic to terminate at the ngrok server, use a TLS or TCP tunnel.

TERMS OF SERVICE: https://ngrok.com/tos

EXAMPLES:
  ngrok http 8080                             # forward ngrok subdomain to port 80
  ngrok http example.com:9000                 # forward traffic to example.com:9000
  ngrok http --domain=bar.ngrok.dev 80        # request subdomain name: 'bar.ngrok.dev'
  ngrok http --domain=example.com 1234        # request tunnel 'example.com' (DNS CNAME)
  ngrok http --basic-auth='falken:joshua' 80  # enforce basic auth on tunnel endpoint
  ngrok http --host-header=example.com 80     # rewrite the Host header to 'example.com'
  ngrok http file:///var/log                  # serve local files in /var/log
  ngrok http https://localhost:8443           # forward to a local https server

OPTIONS:
      --authtoken string                 ngrok.com authtoken identifying a user
      --basic-auth strings               enforce basic auth on tunnel endpoint, 'user:password'
      --cidr-allow strings               reject connections that do not match the given CIDRs
      --cidr-deny strings                reject connections that match the given CIDRs
      --circuit-breaker float            reject requests when 5XX responses exceed this ratio
      --compression                      gzip compress http responses from your web service
      --config strings                   path to config files; they are merged if multiple
      --domain string                    host tunnel on a custom subdomain or hostname (requires DNS CNAME)
  -h, --help                             help for http
      --host-header string               set Host header; if 'rewrite' use local address hostname
      --inspect                          enable/disable http introspection (default true) (default <nil>)
      --log string                       path to log file, 'stdout', 'stderr' or 'false' (default "false")
      --log-format string                log record format: 'term', 'logfmt', 'json' (default "term")
      --log-level string                 logging level: 'debug', 'info', 'warn', 'error', 'crit' (default "info")
      --mutual-tls-cas string            path to TLS certificate authority to verify client certs in mutual tls
      --oauth string                     enforce authentication oauth provider on tunnel endpoint, e.g. 'google'
      --oauth-allow-domain strings       allow only oauth users with these email domains
      --oauth-allow-email strings        allow only oauth users with these emails
      --oauth-client-id string           oauth app client id, optional
      --oauth-client-secret string       oauth app client secret, optional
      --oauth-scope strings              request these oauth scopes when users authenticate
      --oidc string                      oidc issuer url, e.g. https://accounts.google.com
      --oidc-client-id string            oidc app client id
      --oidc-client-secret string        oidc app client secret
      --oidc-scope strings               request these oidc scopes when users authenticate
      --proxy-proto string               version of proxy proto to use with this tunnel, empty if not using
      --region string                    ngrok server region [us, eu, au, ap, sa, jp, in] (default "us")
      --request-header-add strings       header key:value to add to request
      --request-header-remove strings    header field to remove from request if present
      --response-header-add strings      header key:value to add to response
      --response-header-remove strings   header field to remove from response if present
      --scheme strings                   which schemes to listen on (default [https])
      --verify-webhook string            validate webhooks are signed by this provider, e.g. 'slack'
      --verify-webhook-secret string     secret used by provider to sign webhooks, if any
      --websocket-tcp-converter          convert ingress websocket connections to TCP upstream

ERROR:  authentication failed: Your ngrok-agent version "3.3.1" is too old. The minimum supported agent version for your account is "3.20.0". Update to a newer version with 'ngrok update', by downloading from https://ngrok.com/download, or by updating your SDK version. Paid accounts are currently excluded from minimum agent version requirements. To begin handling traffic immediately without updating your agent, upgrade to a paid plan: https://dashboard.ngrok.com/billing/choose-a-plan.
ERROR:
ERROR:  ERR_NGROK_121
ERROR

Show more

You’ve hit an ngrok version gate from your account policy. I’ll upgrade your local agent to the current release and then re-verify ngrok version.

I’ll run the updater via the installed binary path directly (your shell PATH is inconsistent between sessions).

That error was caused by an outdated ngrok agent. It’s now updated on your machine to 3.36.1.

Run this in your current PowerShell session:

$ngrokDir = "$env:LOCALAPPDATA\Microsoft\WinGet\Packages\Ngrok.Ngrok_Microsoft.Winget.Source_8wekyb3d8bbwe"
$env:Path = "$ngrokDir;$env:Path"
ngrok version
Then start tunnel:

ngrok config add-authtoken <YOUR_TOKEN>
ngrok http 5175
If ngrok still isn’t recognized, run directly:

& "$env:LOCALAPPDATA\Microsoft\WinGet\Packages\Ngrok.Ngrok_Microsoft.Winget.Source_8wekyb3d8bbwe\ngrok.exe" http 5175


⚠️ Free Users: Agents ≤3.19.x stop connecting 2/17/26. Update or upgrade: https://ngrok.com/pricing

Session Status                online
Account                       slee7286@gmail.com (Plan: Free)
Version                       3.36.1
Region                        Europe (eu)
Latency                       26ms
Web Interface                 http://127.0.0.1:4042
Forwarding                    https://joya-heliced-jane.ngrok-free.dev -> http://localhost:5175

Connections                   ttl     opn     rt1     rt5     p50     p90
                              1       0       0.01    0.00    6.22    6.22

HTTP Requests                                                                                                           -------------                                                                                                                                                                                                                                   09:43:38.708 GMT GET /                          403 Forbidden                                                           09:43:38.907 GMT GET /favicon.ico               403 Forbidden

Tunnel is healthy; the 403 is almost certainly Vite host-check blocking the ngrok hostname. I’ll quickly check your vite.config.ts and give you the exact fix/command.

ngrok is working; the 403 is from Vite rejecting the ngrok Host header.

Use this first (no code changes):

# keep your dev server running on 5175
ngrok http 5175 --host-header=rewrite
If you still get 403, start Vite with allowed host + explicit host binding:

$env:__VITE_ADDITIONAL_SERVER_ALLOWED_HOSTS="joya-heliced-jane.ngrok-free.dev"
npm run dev -- --host 0.0.0.0 --port 5175
Then run:

ngrok http 5175 --host-header=rewrite
Also make sure ngrok targets the same port your Vite server is actually using (your config default is 5173, unless you override with --port 5175).



