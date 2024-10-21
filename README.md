# BUTTON: Facilitating Multi-turn Function Calling for LLMs via Compositional Instruction Tuning

[![Static Badge](https://img.shields.io/badge/arXiv-2410.12952-b31b1b.svg)](https://arxiv.org/abs/2410.12952)

Large Language Models (LLMs) have exhibited significant potential in performing diverse tasks, including the ability to call functions or use external tools to enhance their performance. While current research on function calling by LLMs primarily focuses on single-turn interactions, this paper addresses the overlooked necessity for LLMs to engage in multi-turn function calling—critical for handling compositional, real-world queries that require planning with functions but not only use functions.

To facilitate this, we introduce an **Bottom-Up Then Top-dOwN** pipeline, ``BUTTON``, which generates synthetic compositional instruction tuning data via bottom-up instruction construction and top-down trajectory generation. 

- **Bottom-Up Phase**: We generate simple atomic tasks based on real-world scenarios and develop compositional tasks using heuristic strategies. Functions are then designed for these compositional tasks.
- **Top-Down Phase**: In a multi-agent environment, interactions among simulated humans, assistants, and tools are used to collect multi-turn function calling trajectories. This ensures task compositionality and allows for effective function and trajectory generation by analyzing atomic tasks within compositional tasks.

We introduce a dataset ``BUTTONInstruct`` comprising 8k data points and demonstrate its effectiveness through extensive experiments across various LLMs.

![metho](./assets/method.png#center)

## 📁 Data Format

``data/button_instruct.jsonl`` is formatted as JSON Lines. Each data point contains a list of messages, and each message is a JSON object with two fields: ``role`` and ``content``.
```json
{
    "messages": [{
        "role": "systems",  // system, user, assistant or tool
        "content": "You are an expert in using functions ... ..."
    }, {
        "role": "user",
        "content": "Hi, I need to find out the estimated delivery date for my package ... ..."
    }, ... ... ]
}
```
There are four roles, ``system``, ``user``, ``assistant``, and ``tool``.

#### system  

The system prompt lists available tools. The template is as follows, with ``{tool_list}`` replaced by available tool descriptions, enclosed within ``<tool></tool>``.

```
You are an expert in using functions (i.e., tools) to solve users' tasks. The functions available for you to use are detailed below: 
<tool>
{tool_list}
</tool>

In your response, you need first provide your observation and thought on the user's task, the current situation, and what you plan to do next. After your thinking, you can do following two things:
**Function Call**: For fountion calling, you need to provide the function name and its arguments. The function name must be same as its name in above function list, and the arguments must obey the format required by the function. Enclose the function call within the tag "<call></call>". {parallel}
**Final Answer**: When you believe the task is complete, you may use 'final_answer' to provide a detailed summary of the results to give to the user, enclose the final answer within the tag "<final></final>".


```
For controlling parallel calling, we replace the ``{parallel}`` in the system prompt with the following two requirements:
- For turning off parallel calling: ``You should call one function at a time, and wait for the response before calling the next function.``
- For turning on parallel calling: ``If possible, you can call multiple functions in parallel, be sure the functions called parallelly are independent of each other.``


#### user
The user role provides the question.

#### assistant
The assistant calls functions iteratively based on the user's question. Each response includes a thought process in natural language, followed by function calls enclosed within ``<call></call>``.

```
User wants to ... ... To address the user's request, we need to ... ... <call>[
    {
        "name": "function_name", 
        "arguments": {"arg_name": "arg_value"}
    },
    ... ...
]</call>
```
Finally, the assistant use ``<final></final>`` to provide a detailed summary of the results to the user.

#### tool
The tool role provides the results of function calls, formatted as a list of JSON objects, each corresponding to a function call from the assistant.

```json
[
    {
        "name": "function_name", 
        "arguments": {"arg_name": "arg_value"},
        "results": {"res_name": "res_value"}
    },
    ... ...
]
```

## 📚 Citation
If you find this work useful, please consider citing our paper:
```bibtex
@article{chen2024facilitating,
    title={Facilitating Multi-turn Function Calling for LLMs via Compositional Instruction Tuning},
    author={Chen, Mingyang and Sun, Haoze and Li, Tianpeng and Yang, Fan and Liang, Hao and Lu, Keer and Cui, Bin and Zhang, Wentao and Zhou, Zenan and Chen, Weipeng},
    journal={arXiv preprint arXiv:2410.12952},
    year={2024}
}
```