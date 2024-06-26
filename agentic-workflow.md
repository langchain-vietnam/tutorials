# Hướng Dẫn Sử Dụng Langgraph Để Xây Dựng Quy Trình Công Việc Tự Động
## Giới Thiệu
Langgraph là một công cụ mạnh mẽ giúp bạn xây dựng và quản lý các quy trình công việc tự động (agentic workflow) một cách dễ dàng và hiệu quả. Trong bài viết này, chúng ta sẽ đi qua các bước cơ bản để sử dụng Langgraph, từ cài đặt đến triển khai một quy trình công việc hoàn chỉnh.

## 1. Cài Đặt Langgraph
Để bắt đầu, bạn cần cài đặt Langgraph. Bạn có thể cài đặt thông qua pip:

```
pip install langgraph
```

## 2. Tạo Một Agent Đơn Giản
Agent là một thực thể có thể thực hiện các tác vụ cụ thể trong quy trình công việc. Dưới đây là ví dụ về cách tạo một agent đơn giản:

```
from langchain import hub
from langchain.agents import create_openai_functions_agent
from langchain_openai.chat_models import ChatOpenAI
from langchain_community.tools.tavily_search import TavilySearchResults
from langgraph.graph import StateGraph

tools = [TavilySearchResults(max_results=1)]

# Chọn câu prompt để sử dụng 
prompt = hub.pull("hwchase17/openai-functions-agent")

# Chọn mô hình LLM cho agent
llm = ChatOpenAI(model="gpt-3.5-turbo-1106", streaming=True)

# Khởi tạo agent
agent_runnable = create_openai_functions_agent(llm, tools, prompt)
```


## 3. Xây Dựng Quy Trình Công Việc (Workflow)
Một workflow là một chuỗi các tác vụ mà các agent thực hiện. Dưới đây là ví dụ về cách xây dựng một workflow cơ bản:

```
from langgraph.graph import END, StateGraph
from typing import TypedDict, Annotated, List, Union
from langchain_core.agents import AgentAction, AgentFinish
from langchain_core.messages import BaseMessage
import operator


class AgentState(TypedDict):
   # dữ liệu đầu vào
   input: str
   # Lịch sử những tin nhắn trước
   chat_history: list[BaseMessage]
   agent_outcome: Union[AgentAction, AgentFinish, None]
   intermediate_steps: Annotated[list[tuple[AgentAction, str]], operator.add]

workflow = StateGraph(AgentState)

```

## 4. Quản Lý Luồng Công Việc Với Langgraph
Langgraph cho phép bạn quản lý các luồng công việc một cách hiệu quả thông qua việc định nghĩa các bước và các tác vụ tương ứng. Bạn có thể thêm, xóa hoặc sửa đổi các bước trong quy trình công việc của mình.

Tạo ra điểm bắt đầu của luồng quản lí công việc
```
workflow.set_entry_point("agent")
```

Thêm Bước vào Workflow
```
workflow.add_edge('action', 'agent')
workflow.add_node("agent", run_agent)
workflow.add_node("action", execute_tools)
workflow.add_conditional_edges(
    # First, we define the start node. We use `agent`.
    # This means these are the edges taken after the `agent` node is called.
    "agent",
    # Next, we pass in the function that will determine which node is called next.
    should_continue,
    # Finally we pass in a mapping.
    # The keys are strings, and the values are other nodes.
    # END is a special node marking that the graph should finish.
    # What will happen is we will call `should_continue`, and then the output of that
    # will be matched against the keys in this mapping.
    # Based on which one it matches, that node will then be called.
    {
        # If `tools`, then we call the tool node.
        "continue": "action",
        # Otherwise we finish.
        "end": END
    }
)
```

## 5. Triển Khai Quy Trình Công Việc
Sau khi xây dựng và kiểm tra quy trình công việc, bạn có thể triển khai nó vào môi trường thực tế. Điều này có thể được thực hiện bằng cách tích hợp Langgraph vào các ứng dụng hoặc hệ thống của bạn.

```
app = workflow.compile()
```

# Kết Luận
Langgraph là một công cụ hữu ích cho việc xây dựng và quản lý các quy trình công việc tự động. Với khả năng tạo và quản lý các agent một cách linh hoạt, bạn có thể dễ dàng xây dựng các workflow phức tạp để tự động hóa nhiều tác vụ khác nhau trong ứng dụng của mình. Hy vọng hướng dẫn này sẽ giúp bạn bắt đầu với Langgraph và xây dựng những quy trình công việc hiệu quả.
