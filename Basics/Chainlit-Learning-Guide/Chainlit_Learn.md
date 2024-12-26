# Chainlit Learning Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Installation Guide](#installation-guide)
4. [Running Applications](#running-applications)
5. [Core Concepts](#core-concepts)
   - [Basic Setup](#1-basic-setup)
   - [Message Handling](#2-message-handling)
   - [State Management](#3-state-management)
   - [UI Components](#4-ui-components)
   - [Progress Tracking](#5-progress-tracking)
   - [Interactive Elements](#6-interactive-elements)
   - [File Operations](#7-file-operations)
   - [Error Handling](#8-error-handling)
   - [Async Integration with External APIs](#9-async-integration-with-external-apis)
6. [Best Practices](#best-practices)
7. [Additional Resources](#additional-resources)

## Introduction
Chainlit is a Python library for building interactive chat applications. It provides a simple API for creating chatbots, conversational interfaces, and interactive messaging applications with features like complex chat workflows, state management, and rich UI elements.

## Prerequisites
- Python 3.7+ (recommended: Python 3.9 or 3.10)
- pip package manager

## Installation Guide

### Standard Installation
```bash
pip install chainlit
```

### Troubleshooting Installation
If you encounter installation issues, follow these steps:
1. Uninstall existing installations:
```bash
pip uninstall chainlit pydantic
```

2. Install specific versions known to work:
```bash
pip install pydantic==1.10.12
pip install chainlit==0.7.700
```

3. If issues persist:
```bash
pip cache purge
```

## Running Applications
Start your Chainlit application:
```bash
# Get help and see all available options
chainlit run --help

# Basic run
chainlit run app.py

# With auto-reload for development
chainlit run app.py -w

# Run on a specific port (default is 8000)
chainlit run app.py -w --port 3001
```

## Core Concepts

### 1. Basic Setup
Create a simple "Hello World" application:
```python
# Import the Chainlit library
import chainlit as cl

# Decorator that marks a function to be called when a new chat session starts
@cl.on_chat_start
async def start():
    # Send a simple message to the chat interface
    # The await keyword is required as this is an asynchronous operation
    await cl.Message(content="Hello world!").send()
```

### 2. Message Handling
```python
@cl.on_message  # Decorator to handle all incoming chat messages
async def main(message: cl.Message):
    # message.content contains the text sent by the user
    user_input = message.content
    
    # Create and send a response message
    # author parameter shows who sent the message in the chat interface
    await cl.Message(
        content=f"Received: {user_input}",
        author="Assistant"
    ).send()
```

### 3. State Management
```python
# Global dictionary to store conversation history for multiple users
conversation_history = {}

@cl.on_chat_start
async def start():
    # Get unique identifier for current user session
    session_id = cl.user_session.get("id")
    
    # Initialize empty list for this user's conversation
    # Each user gets their own conversation history
    conversation_history[session_id] = []
```

### 4. UI Components
#### Messages
```python
# Send a message with additional UI elements and actions
await cl.Message(
    content="Main content",          # The primary message text
    author="Assistant",              # Who sent the message
    elements=[],                     # List of UI elements (images, text blocks, etc.)
    actions=[]                       # Interactive elements like buttons
).send()
```

#### Images
```python
# Display an image in the chat
await cl.Image(
    path="./image.png",             # Local path to image file
    name="unique_name",             # Unique identifier for the image
    display="inline"                # How the image should be displayed
).send()
```

#### Text Elements
```python
# Display formatted text with syntax highlighting
await cl.Text(
    content="Formatted text",        # The text content to display
    name="text_element",            # Unique identifier
    language="python"               # Programming language for syntax highlighting
).send()
```

### 5. Progress Tracking
```python
# Create a progress indicator for long-running operations
async with cl.Step(name="Processing", show_progress=True) as step:
    # Your long-running task here
    
    # Update the step's status when complete
    await step.update(name="Completed!")
```

### 6. Interactive Elements
```python
# Decorator to handle button clicks
# button_id matches the name parameter in cl.Action
@cl.action_callback("button_id")
async def on_action(action):
    # action.value contains the value passed from the button
    await cl.Message(content=f"Button {action.value} clicked").send()

# Create an interactive button
actions = [
    cl.Action(
        name="button_id",            # Identifier for the button
        value="action_value",        # Value passed to callback when clicked
        label="Click Me"             # Text shown on the button
    )
]
```

### 7. File Operations
```python
@cl.on_chat_start
async def start():
    # Prompt user to upload a file
    # accept parameter specifies allowed file types
    files = await cl.AskFileMessage(
        content="Upload a file",
        accept=["text/plain", "application/pdf"]
    ).send()
    
    # Check if files were uploaded and process them
    if files:
        file = files[0]  # Get the first uploaded file
        # Process file content here
```

### 8. Error Handling
```python
try:
    # Attempt potentially risky operations
    result = process_data()
except Exception as e:
    # Send error message to chat with system author tag
    await cl.Message(
        content=f"Error: {str(e)}",
        author="System"              # Indicates system message
    ).send()
```

### 9. Async Integration with External APIs
```python
import chainlit as cl
import aiohttp

@cl.on_message
async def main(message: cl.Message):
    # Show a loading message while fetching data
    async with cl.Step("Fetching weather data..."):
        try:
            # Make a simple API call
            async with aiohttp.ClientSession() as session:
                # Example: Getting weather data for London
                url = "https://api.weatherapi.com/v1/current.json"
                params = {
                    "key": "YOUR_API_KEY",
                    "q": "London"
                }
                
                async with session.get(url, params=params) as response:
                    if response.status == 200:
                        data = await response.json()
                        # Send the weather information to the chat
                        await cl.Message(
                            content=f"Current temperature in London: {data['current']['temp_c']}°C"
                        ).send()
                    else:
                        raise Exception(f"API returned status code: {response.status}")
                        
        except Exception as e:
            # Handle any errors
            await cl.Message(
                content=f"Sorry, I couldn't fetch the weather data: {str(e)}",
                author="System"
            ).send()
```

## Best Practices
1. Always use async/await for Chainlit functions
2. Implement proper error handling
3. Use session management for user-specific data
4. Structure your code using decorators
5. Provide clear feedback to users

## Additional Resources
- [Official Chainlit Documentation](https://docs.chainlit.io)
- [GitHub Repository](https://github.com/Chainlit/chainlit)