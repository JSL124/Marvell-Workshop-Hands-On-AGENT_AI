# 🌍 Multi-Agent Travel Planner Workshop

A **progressive, hands-on workshop** that takes you from a basic LLM chatbot to a production-ready multi-agent system. Built with Python, Streamlit, **LangGraph**, **ReAct Framework**, and Groq's Llama 3.1 model.

## 🎯 Workshop Format

This is a **build-as-you-learn** workshop! We start simple and add complexity step-by-step:

```
Basic LLM → Single Agent → Multi-Agent → LangGraph → ReAct → Complete System
  (15min)     (20min)       (25min)       (30min)    (25min)    (15min)
```

### 📚 Workshop Stages

1. **Stage 1**: Basic LLM Chatbot - See the problems
2. **Stage 2**: First Specialized Agent - See the improvement  
3. **Stage 3**: Multi-Agent System - See coordination challenges
4. **Stage 4**: LangGraph Workflow - Professional orchestration
5. **Stage 5**: ReAct Framework - Transparent reasoning
6. **Stage 6**: Complete System - Production-ready!

**📖 Detailed Workshop Guide**: See [WORKSHOP_STRUCTURE.md](WORKSHOP_STRUCTURE.md) for stage-by-stage instructions.

## 💡 What You'll Learn

By building progressively, you'll understand **why** each component exists:

- ✅ When a basic LLM isn't enough
- ✅ How specialized agents add value
- ✅ Why workflow orchestration matters
- ✅ How to make AI decisions transparent
- ✅ Production-ready multi-agent patterns

## 🏗️ Architecture

### LangGraph Workflow Architecture

```
                    START
                      ↓
              [Parse Request Node]
                      ↓
              [Flight Agent Node]
                      ↓
          [Accommodation Agent Node]
                      ↓
            [Itinerary Agent Node]
                      ↓
             [Budget Agent Node]
                      ↓
            [Summary Generator Node]
                      ↓
                     END
```

**State Management**: Each node reads from and writes to a shared `TravelPlanState` that flows through the graph. LangGraph handles state transitions automatically.

**ReAct Reasoning**: Every agent uses the ReAct pattern (Thought → Action → Observation) to make decisions transparent and debuggable.

### The Six Workflow Nodes

#### 1. **Parse Request Node** (`parse_request_node`)
- **Role:** Entry point for the workflow
- **Responsibilities:**
  - Extracts structured data from natural language input
  - Populates initial state (origin, destination, budget, preferences)
  - Uses LLM for intelligent parsing
- **State Updates:** `origin`, `destination`, `budget`, `days`, `nights`, `preferences`

#### 2. **Flight Agent Node** (`flight_agent_node`)
- **Role:** Flight recommendation
- **Responsibilities:**
  - Filters flights by origin, destination, and budget
  - Evaluates options based on price, duration, and comfort
  - Provides reasoning for recommendations
- **State Updates:** `flight_result`

#### 3. **Accommodation Agent Node** (`accommodation_agent_node`)
- **Role:** Lodging recommendation
- **Responsibilities:**
  - Filters accommodations by city and remaining budget
  - Balances location, amenities, and cost
  - Calculates total stay costs
- **State Updates:** `accommodation_result`

#### 4. **Itinerary Agent Node** (`itinerary_agent_node`)
- **Role:** Activity planning
- **Responsibilities:**
  - Selects attractions matching destination
  - Balances activity types (culture, nature, food)
  - Creates day-by-day schedules
- **State Updates:** `itinerary_result`

#### 5. **Budget Agent Node** (`budget_agent_node`)
- **Role:** Financial validation
- **Responsibilities:**
  - Performs deterministic cost calculations
  - Validates against total budget
  - Suggests optimizations if over-budget
- **State Updates:** `budget_result`

#### 6. **Summary Generator Node** (`summary_node`)
- **Role:** Final synthesis
- **Responsibilities:**
  - Aggregates all agent outputs
  - Generates executive summary using LLM
  - Determines overall success status
- **State Updates:** `summary`, `success`

### Why LangGraph + ReAct?

**LangGraph provides:**
- **Explicit State Management**: Shared state object passed through all nodes
- **Visual Workflow**: Clear DAG structure makes agent flow easy to understand
- **Extensibility**: Easy to add conditional edges, parallel execution, or retry logic
- **Type Safety**: TypedDict ensures state schema consistency
- **Debugging**: Can inspect state at each node transition

**ReAct Framework provides:**
- **Transparent Reasoning**: See how agents think (Thought)
- **Clear Actions**: Understand what agents decide (Action)
- **Insight Generation**: Learn from agent observations (Observation)
- **Debuggable Decisions**: Trace why recommendations were made
- **Trust Building**: Users can verify agent reasoning

## 🔄 Understanding the Architecture

### The Journey from Simple to Sophisticated

**Stage 1 - Basic LLM:**
```python
response = llm("Plan a trip to Tokyo")
# ❌ Makes up data, no reasoning shown
```

**Stage 2 - Specialized Agent:**
```python
flight = flight_agent(real_data, budget)
# ✅ Uses real data, applies expertise
```

**Stage 3 - Multi-Agent:**
```python
flight = flight_agent()
accommodation = accommodation_agent(remaining_budget)
# ✅ Coordination, but hard-coded
```

**Stage 4 - LangGraph:**
```python
workflow = StateGraph(TravelState)
workflow.add_node("flight", flight_node)
workflow.add_edge("flight", "accommodation")
# ✅ Visual workflow, explicit state
```

**Stage 5 - ReAct:**
```python
💭 Thought: Analyze options...
🎯 Action: Recommend specific choice
👁️ Observation: Explain trade-offs
# ✅ Transparent reasoning!
```

**Stage 6 - Complete System:**
All of the above working together! 🎉

### State Schema

The `TravelPlanState` TypedDict flows through all nodes:

```python
class TravelPlanState(TypedDict):
    # Input from user
    user_input: str
    origin: str
    destination: str
    budget: float
    days: int
    nights: int
    preferences: str
    
    # Agent outputs
    flight_result: Dict
    accommodation_result: Dict
    itinerary_result: Dict
    budget_result: Dict
    
    # Final output
    summary: str
    success: bool
```

### Execution Flow

1. **START** → User submits travel request
2. **Parse Node** → Extracts structured parameters from natural language
3. **Flight Node** → Recommends flights, updates `flight_result` in state
4. **Accommodation Node** → Reads `flight_result`, recommends lodging
5. **Itinerary Node** → Reads previous results, creates daily plans
6. **Budget Node** → Validates all costs, flags issues
7. **Summary Node** → Generates final summary using all results
8. **END** → Returns complete plan to UI

**Key Insight**: Each node is a **pure function** that takes state and returns updated state. Nodes don't call each other—LangGraph handles transitions.

### Code Example

```python
# Each node has this signature:
def flight_agent_node(state: TravelPlanState) -> TravelPlanState:
    """Read from state, do work, return updated state."""
    flight_result = recommend_flight(
        origin=state["origin"],
        destination=state["destination"],
        budget=state["budget"] * 0.4
    )
    return {**state, "flight_result": flight_result}

# Graph definition:
workflow = StateGraph(TravelPlanState)
workflow.add_node("flight_agent", flight_agent_node)
workflow.add_edge("parse_request", "flight_agent")  # Sequential
app = workflow.compile()

# Execution:
final_state = app.invoke(initial_state)  # LangGraph handles the rest!
```

## 🌟 What Makes This Special?

This workshop combines **three cutting-edge patterns**:

1. **🔄 LangGraph**: Explicit state management with visual workflow graphs
2. **🧠 ReAct Framework**: Transparent reasoning (Thought → Action → Observation)
3. **🤖 Multi-Agent**: Specialized agents collaborating without direct communication

**Result**: A production-ready pattern that's transparent, debuggable, and extensible!

### Example: Flight Agent Using ReAct

```python
💭 Thought: User needs Singapore to Tokyo flight. Budget is $600. 
            I see options from $320-$680. Need to balance cost and convenience.

🎯 Action: I recommend Flight FL001 by SkyHigh Airlines at $450.
           Departs 08:30, arrives 16:00, direct flight, 6.5 hours.

👁️ Observation: This choice is mid-range price but provides excellent morning 
                 departure time. Saves $130 from the business class option while 
                 maintaining convenience. The $320 option exists but departs at 14:00.
```

**Why this matters**: You can see *exactly* how the agent thinks, not just what it recommends!

## 🚀 Two Ways to Use This Workshop

### Option 1: Follow the Progressive Workshop (Recommended for Learning)

**Best for**: Understanding WHY each component exists

1. **Setup** (5 min):
   ```bash
   git clone <your-repo-url>
   cd nushacknroll-marvell
   python -m venv venv
   venv\Scripts\activate  # Windows
   pip install -r requirements.txt
   ```

2. **Configure API key**:
   ```bash
   Copy-Item env.example .env  # Windows
   # Edit .env and add your Groq API key
   ```

3. **Follow the stages**:
   - Read [WORKSHOP_STRUCTURE.md](WORKSHOP_STRUCTURE.md)
   - Start with Stage 1 (Basic LLM)
   - Progress through each stage
   - See problems → solutions → improvements

### Option 2: Run the Complete System (For Quick Demo)

**Best for**: Seeing the end result first

1. **Quick setup**:
   ```bash
   pip install -r requirements.txt
   Copy-Item env.example .env
   # Add your API key to .env
   ```

2. **Run the app**:
   ```bash
   streamlit run app.py
   ```

3. **Explore the features**:
   - Try different travel requests
   - Click on "🧠 Agent Reasoning" to see ReAct thinking
   - Check the Budget Analysis tab

4. **Then go back** and follow the progressive workshop to understand how it works!

---

## 📋 Prerequisites

- Python 3.10 or higher
- A Groq API key ([Get free key](https://console.groq.com/keys))
- Basic Python knowledge
- (Optional) Jupyter for interactive notebooks

## 📝 How to Use

1. **Open the app** in your browser
2. **Describe your trip** in natural language, for example:
   - "I want to visit Tokyo for 4 days with a budget of $1500. I love culture and food."
   - "Plan a 5-day trip to Paris with $2000. Prefer central locations."
   - "Beach vacation in Bali for 3 days, budget $800."
3. **Click "Generate Travel Plan"**
4. **Review the results** across four tabs:
   - ✈️ **Flight:** Recommended flight with reasoning
   - 🏨 **Accommodation:** Hotel/lodging suggestion
   - 📅 **Itinerary:** Day-by-day activity plan
   - 💰 **Budget:** Cost breakdown and validation

## 📂 Project Structure

```
nushacknroll-marvell/
├── app.py                      # Streamlit UI entry point
├── agents/                     # All agent modules
│   ├── supervisor.py           # Orchestration agent (LangGraph workflow)
│   ├── flight_agent.py         # Flight recommendations (ReAct)
│   ├── accommodation_agent.py  # Lodging recommendations (ReAct)
│   ├── itinerary_agent.py      # Activity planning (ReAct)
│   └── budget_agent.py         # Cost validation (ReAct)
├── data/                       # Mock JSON data
│   ├── flights.json            # 12 sample flights
│   ├── accommodations.json     # 10 sample hotels
│   └── attractions.json        # 15 sample activities
├── prompts/                    # LLM prompt templates (ReAct format)
│   ├── supervisor.txt
│   ├── flight_agent.txt
│   ├── accommodation_agent.txt
│   ├── itinerary_agent.txt
│   └── budget_agent.txt
├── utils/                      # Shared utilities
│   ├── llm.py                  # LLM wrapper with ReAct support
│   └── data_loader.py          # JSON data loader
├── requirements.txt            # Python dependencies
├── env.example                 # Environment template
├── .env                        # Your actual API key (create this!)
├── README.md                   # This file
├── LANGGRAPH_GUIDE.md          # LangGraph deep dive
├── REACT_FRAMEWORK.md          # ReAct framework guide
├── WORKSHOP_OVERVIEW.md        # Complete workshop guide
└── SETUP_INSTRUCTIONS.md       # Quick setup guide
```

## 🔧 Technical Details

### Tech Stack

- **Python 3.10+**: Core language
- **Streamlit**: Web UI framework
- **LangGraph**: State machine workflow orchestration
- **ReAct Framework**: Transparent reasoning pattern (Thought → Action → Observation)
- **LangChain**: LLM utilities and abstractions
- **Groq API**: Fast LLM inference (Llama 3.1 8B)
- **JSON**: Mock data storage (no external APIs)

### LLM Configuration

- **Model:** `llama-3.1-8b-instant`
- **Temperature:** 0.2 (low for consistent, logical outputs)
- **Max Tokens:** 1024 (keeps responses concise)
- **Strategy:** Short, focused prompts per agent

### Design Patterns

1. **ReAct Pattern:** Thought → Action → Observation for all agent decisions
2. **StateGraph Pattern:** LangGraph manages workflow as a directed acyclic graph (DAG)
3. **Agent-as-Node Pattern:** Each agent is a stateless node function
4. **Shared State Pattern:** TypedDict state flows through all nodes
5. **Singleton Pattern:** Shared LLM and data loader instances
6. **Prompt Templates:** Externalized in `.txt` files for easy modification
7. **Separation of Concerns:** UI, agents, data, and utilities are decoupled

## 🎓 Extension Ideas for Participants

### Beginner Extensions
- [ ] Add more destinations and data to JSON files
- [ ] Implement date filtering for flights
- [ ] Add hotel type preferences (budget/luxury)
- [ ] Include free activities in itineraries
- [ ] Improve ReAct parsing with better regex patterns

### Intermediate Extensions
- [ ] Add conditional edges in LangGraph (e.g., retry if budget fails)
- [ ] Implement parallel node execution for independent agents
- [ ] Add a "Review Agent" node that critiques the plan
- [ ] Create a feedback loop where Budget Agent can trigger re-planning
- [ ] Add weather considerations
- [ ] Implement multi-city trip support with sub-graphs

### Advanced Extensions
- [ ] Replace mock data with real APIs (Amadeus, Booking.com)
- [ ] Add LangGraph visualization using Mermaid diagrams
- [ ] Implement human-in-the-loop approval nodes
- [ ] Add checkpointing for workflow persistence
- [ ] Create streaming responses from each node
- [ ] Add error handling nodes with retry logic
- [ ] Implement A/B testing with multiple workflow paths

## 🐛 Troubleshooting

### "GROQ_API_KEY not found"
- Ensure you created a `.env` file (not `env.example`)
- Check that your API key is correctly formatted in `.env`
- Verify the `.env` file is in the project root directory (same folder as `app.py`)
- See [SETUP_INSTRUCTIONS.md](SETUP_INSTRUCTIONS.md) for detailed setup

### "No module named 'groq'"
- Run `pip install -r requirements.txt`
- Ensure your virtual environment is activated

### "No flights/accommodations found"
- Check that destination names match the JSON data (case-insensitive)
- Available destinations: Tokyo, Paris, Bali, Bangkok, New York, Sydney
- Adjust budget constraints if too restrictive

### App won't start
- Ensure you're in the correct directory: `cd travel-agent-workshop`
- Check Python version: `python --version` (should be 3.10+)
- Try: `streamlit run app.py --server.headless true`

## 📚 Learning Resources

### ReAct Framework
- [ReAct Paper (Original)](https://arxiv.org/abs/2210.03629)
- [LangChain ReAct Agents](https://python.langchain.com/docs/modules/agents/agent_types/react)
- [Our ReAct Guide](REACT_FRAMEWORK.md) - Detailed implementation guide

### Multi-Agent Systems
- [LangChain Agents Documentation](https://python.langchain.com/docs/modules/agents/)
- [LangGraph Tutorial](https://langchain-ai.github.io/langgraph/)
- [Our LangGraph Guide](LANGGRAPH_GUIDE.md) - Workshop-specific guide
- [Multi-Agent Design Patterns (Paper)](https://arxiv.org/abs/2308.08155)

### Prompt Engineering
- [Groq Documentation](https://console.groq.com/docs)
- [Prompt Engineering Guide](https://www.promptingguide.ai/)
- [ReAct Prompting](https://www.promptingguide.ai/techniques/react)

### Streamlit
- [Streamlit Documentation](https://docs.streamlit.io/)
- [Streamlit Gallery](https://streamlit.io/gallery)

## 🤝 Contributing

This is a workshop project! Feel free to:
- Add more agents (e.g., Weather Agent, Translation Agent)
- Improve prompt engineering
- Enhance the UI
- Add error handling
- Create unit tests

## 📄 License

This project is provided as educational material for workshop purposes. Feel free to use and modify as needed.

## 🙏 Acknowledgments

- **Groq** for providing fast, free LLM inference
- **Streamlit** for making web UIs simple
- **LangChain** for agent orchestration utilities

---

**Workshop Contact:** For questions during the workshop, ask your instructor or check the project Issues tab.

**Happy Building! 🚀**

