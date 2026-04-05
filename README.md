AgentTap
A real-time debugging proxy for Agent2Agent (A2A) multi-agent systems.

AgentTap sits between your client agent and any remote A2A agent, intercepts every JSON-RPC message, and surfaces the full request/response traffic in a live dashboard — so you can finally see what's on the wire.

Built for developers who are tired of print() debugging their agent pipelines.

Why AgentTap
Debugging A2A agents today means guessing. You send a task, something fails, and you have no idea whether the problem is in your client, the remote agent, the payload format, or the network. AgentTap gives you a Wireshark-style view of your entire agent communication, without changing a single line of your code.

Zero code changes — point your client at the proxy instead of the agent. That's it.
Real-time dashboard — see requests and responses as they happen, with JSON syntax highlighting.
Task timeline — group all messages by task_id and follow a multi-turn conversation from start to finish.
SSE streaming support — intercepts both standard HTTP and Server-Sent Events streaming responses.
Error visibility — connection failures, timeouts, and malformed requests are captured and surfaced clearly.
Quick Start
# 1. Clone
git clone https://github.com/your-org/agenttap
cd agenttap

# 2. Create virtualenv and install dependencies
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 3. Start everything (proxy + demo agent + dashboard)
bash start.sh
Open http://localhost:3000 in your browser.

That's it. You're intercepting A2A traffic.

How It Works
Your Client Agent
      │
      ▼
AgentTap Proxy :8080  ◄──── Dashboard :3000
      │                      (WebSocket /ws)
      ▼
Remote Agent :10000
Your client agent sends requests to :8080 exactly as it would to :10000. AgentTap intercepts, logs, and forwards — fully transparent. The dashboard connects via WebSocket and receives every message in real time.

Configuration
Copy .env.example to .env to customize:

cp .env.example .env
Variable	Default	Description
PROXY_PORT	8080	Port the proxy listens on
TARGET_URL	http://localhost:10000	The A2A agent to forward traffic to
DASHBOARD_PORT	3000	Port for the web dashboard
MAX_MESSAGES	500	Max messages kept in memory (ring buffer)
Start Services Manually
If you prefer to control each service separately:

# Terminal 1 — Demo echo agent (or use your own)
python simple_echo_agent.py

# Terminal 2 — Proxy
PROXY_PORT=8080 TARGET_URL=http://localhost:10000 python proxy.py

# Terminal 3 — Dashboard
python -m http.server 3000
Dashboard
┌─────────────────┬──────────────────────┬─────────────────┐
│ Live Traffic    │  Message Detail      │  Task Timeline  │
│                 │                      │                 │
│ ▶ POST /        │  [Request][Response] │  ○ 12:00:01     │
│   task_id: abc  │                      │  ● 12:00:02     │
│   completed     │  Headers             │  ● 12:00:07     │
│                 │  Body (JSON)         │                 │
└─────────────────┴──────────────────────┴─────────────────┘
Left panel — live list of intercepted requests, newest first. Click any row to inspect it.

Center panel — full request and response detail: headers, body, status code, latency.

Right panel — task timeline. Every message sharing the same task_id is grouped here. This is the killer feature for debugging multi-turn task flows where a single logical operation spans multiple HTTP exchanges.

Proxy API
The proxy exposes a small REST API for programmatic access to captured traffic:

Method	Path	Description
GET	/health	Health check + message count + target
GET	/messages	All captured messages as JSON array
DELETE	/messages	Clear message history
WS	/ws	Real-time message stream
*	/*	Transparent proxy to TARGET_URL
Running the Tests
The included test client sends 6 A2A tasks through the proxy — including a multi-turn conversation on the same task_id and an intentionally malformed request — so you can see AgentTap in action immediately.

python test_client.py
Open http://localhost:3000 while the tests run to watch the traffic appear live.

Demo Agents
Two lightweight A2A-compatible agents are included for local testing:

Agent	Port	Description
simple_echo_agent.py	10000	Echoes any message back. No streaming.
slow_agent.py	10001	Simulates a long-running task with submitted → working → completed transitions via SSE streaming.
Point AgentTap at either one to explore different traffic patterns:

TARGET_URL=http://localhost:10001 python proxy.py
Use AgentTap With Your Own Agent
No code changes needed. Just redirect your A2A client to the proxy:

# Before
client = A2AClient("http://my-agent.internal:8000")

# After — add AgentTap in between
client = A2AClient("http://localhost:8080")

# Set TARGET_URL=http://my-agent.internal:8000 when starting the proxy
Everything your client sends will be captured, forwarded, and displayed. Remove AgentTap by pointing back to the original URL.

Stack
Python 3.11+
FastAPI — proxy server and REST API
httpx — async HTTP client with connection pooling
uvicorn — ASGI server
Vanilla JS — dashboard (no build step, no dependencies)
Roadmap
 pip install agenttap — single command install
 HAR export — open captured sessions directly in Chrome DevTools or Burp Suite
 Message replay — re-send any captured request with one click
 gRPC support — intercept A2A traffic over gRPC (A2A spec v0.3+)
 Persistent sessions — survive proxy restarts, share sessions with teammates
 Spec validation — flag messages that don't conform to the A2A spec
Contributing
AgentTap is early and moving fast. Bug reports, feature requests, and pull requests are very welcome.

If you're building with A2A and hit a debugging pain point this tool doesn't solve — open an issue. That's exactly the kind of feedback that shapes what we build next.

License
MIT
