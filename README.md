# simple-multi-agent [WIP]
A simple event driven multi agent framework.

Usually, a relatively complex task needs to be decomposed into multiple subtasks or completed by multiple roles (Agents). 
It is usually cumbersome to write the code logic for multi-roles cooperation.
Every time a new task requires multiple roles to cooperate, it is often necessary to write a task decomposition, 
routing, parallel, aggregation and other logic.

In addition, the existing Agent frameworks are usually very complex, 
the semantics and logic are not concise enough, which it is difficult to use and understand.

This framework provides a simple way to quickly implement multi-agent task scheduling.

## Features
1. Scalable Multi Agents Task Orchestration. (WIP).
2. Auto Parallel Execution. (WIP).
3. Autonomous Execution Mode. (TODO).
4. Orchestrate by Config. (TODO).
5. More...

## Install

TODO, NOT SUPPORTED YET...

## Usage

**1. Define Agents and Auto Orchestration.**

```python
# Setup os environ params
os.environ['OPENAI_API_KEY'] = ...

# Define a junior translator agent.
# This agent receive translation tasks, translate, and submit translation results.
# This agent also receive review results from senior translator, then modify translation results 
# according to review results, then resubmit.
junior_translator_instruction = Instruction(role="translator", goal="translate Chinese to English.", 
                                            note="try to use natural language for translation")
junior_translator = Agent(name=junior_translator, instruction=junior_translator_instruction, 
                          listen=["TranslationTaskEvent", "TranslationReviewSubmitEvent"], 
                          send=["TranlationResultSubmitEvent"], process_function=do_translate)

# Define a senior translator agent receive translation results and review them.
senior_translator_instruction = Instruction(desc="""You are a senior translation reviewer, your goal is 
review translation result from a junior translator. If the translation result passes your check, just say "PASSED".
If the translation result has problems, you should point out the specific problems.
Review rules:
- Accurately convey news facts and background when translating.
- Do not translate names.
- Also retain the cited papers, such as [20]."""
)
senior_translator = Agent(name=senior_translator, instruction=senior_translator_instruction, model="gpt-4-turbo", 
                          listen=["TranslationResultSubmitEvent"], send=["TranslationReviewSubmitEvent"])

# Construct execution environment.
environment = Environment(agents=[junior_translator, senior_translator])
```

**2. User Defined Function**
```python
def do_translate(context):
    # check translation review result.
    if context.last_output.text == 'PASSED':
        context.finish(result=context.current_agent.last_output.text)
        return 
    # do translation
    context.current_agent.execute()
```

**3. Execute a Task.**

```python
event = BaseEvent(name="TranslationTaskEvent", text="你好，世界！")
executor = Executor(environment, event)
result = executor.run()
```

```
result.output

>>> Hello, World!
```

```
result.trace

>>> Main Event: TranslationTaskEvent(text="你好，世界！"), Received Agent: [senior_translator].
>>> Agent Execute Log: name=senior_translator, result=Hello, World!
>>> Event: TranslationResultSubmitEvent(from=junior_translator, text="Hello, World!"), Received Agent: [senior_translator].
>>> Agent Execute Log: name=senior_translator, result=PASSED
>>> Event: TranslationReviewSubmitEvent(from=senior_translator, text="PASSED"), Received Agent: [junior_translator].
>>> Task Finished. Final Result: Hello, World!
```