# Copilot Studio Governance Summary

## M365 Copilot users – what they can do

- Use Copilot in Word, Excel, Teams, etc.
- Interact with agents (chat/use them) 
- Create basic/light agents (limited) 
- Cannot fully manage/publish enterprise agents without Copilot Studio

---

## Copilot Studio: User Licence vs Copilot Studio (capacity)

### Copilot Studio (capacity / trial)

- Provides AI credits (messages/sessions)
- Enables bots to run and respond
- Tenant-level (shared across users)

### Copilot Studio User Licence

- Gives user access to UI
- Allows building/editing
- Does NOT provide capacity on its own

---

## Who can publish agents in Copilot Studio

- Users with Environment Maker role
- AND access to the environment  
- AND tenant has capacity

Licence alone does NOT control publishing

---

## Default environment (access + publishing)

- Accessible to all users by default 
- Cannot restrict with security group
- Anyone with Environment Maker can publish bots
- Main governance risk if not locked down

---

## What uses credits (Copilot Studio)

### Uses credits

- User chatting with bot
- AI-generated responses
- Testing bots
- Published bot usage (Teams/web/etc.)

### Does NOT use credits

- Creating bots
- Editing bots
- Publishing bots
- Opening Copilot Studio

---

## Credits: User licence vs Copilot Studio

- Copilot Studio (capacity): Uses credits
- User Licence only: Does NOT use credits

---

## Summary

- Capacity = what gets consumed
- Roles + environments = who can build/publish
- Default environment = open unless locked down

# Copilot Studio – Publishing, Access, and Credit Usage

## Publishing a Bot

- Publishing a bot does NOT use credits
- Deploying to Teams or Web does NOT use credits
- Credits are only used when someone interacts with the bot

---

## Who Can Interact With a Bot

### 🔹 Not Shared

- Only the creator can access  
- Credits used when creator tests the bot

---

### 🔹 Shared with Specific Users

- Only selected users can interact  
- Credits used when those users chat 

---

### 🔹 Shared via Teams / Channel

- Anyone in the Team can interact  
- Credits used by all users chatting 

---

### 🔹 Shared via Web / Link

- Anyone with access can use it  
- Credits used per interaction 

---

## Examples of Credit Usage

### 🔹 Example 1 – Testing (Low Usage)

- 1 user testing bot  
- 3–5 questions asked  
- Small credit usage  

---

### 🔹 Example 2 – Pilot Users (Medium Usage)

- 5 users × 10 questions  
- = 50 interactions  
- Moderate credit usage  

---

### 🔹 Example 3 – Teams Rollout (High Usage)

- 50 users × 5 questions  
- = 250 interactions  
- High credit usage  

---

### 🔹 Example 4 – Generative AI (Higher Cost)

- Uses knowledge sources (SharePoint, files, web)  
- Each response consumes more credits  
- Faster credit consumption  

---

## What DOES NOT Use Credits

- Creating a bot
- Editing a bot
- Publishing a bot
- Opening Copilot Studio

---

## What DOES Use Credits

- User chatting with the bot
- AI-generated responses
- Testing in Copilot Studio
- Published bot usage (Teams, Web, etc.)

---

## ummary

- Bots are **free to create and publish**  
- Credits are only used when **someone chats with the bot**  
- More users + more questions = **more credit usage**  

---
