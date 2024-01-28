# Local Voice assistant on HA

Reference https://www.home-assistant.io/voice_control/voice_remote_local_assistant

1. Install the Whisper, Piper, openWakeWord add-ons
2. Start the add-ons
3. Open Settings > Devices & Services and in Discovered section press "Configure" for each Whisper, Piper, openWakeWord
4. For each add-on press "Submit" then "Finish"
5. For each add-on press "Start"
6. Open Settings > Voice assistants and select "Add assistant"
7. Enter a name and select language
8. Under Conversation agent, select "Home Assistant".
9. Under Speech-to-text, select "faster-whisper".
10. Under Text-to-speech, select "piper"
11. Under Wakeword engine, select "openwakeword"
12. Under Wakeword, select "ok nabu"
13. Press "Create"
14. Go to Settings -> Devices & Services -> Helpers
15. Click the Create Helper button and select the Toggle option in the list
16. Enter some name, choose icon "mdi:toggle-switch" and press "Create"
17. Go to Settings > Voice assistants. Open the Expose tab.
18. Select Expose entities and toggle "Expose" and add alias "Switch" on just created entity
19. Open Overview and touch Assist button
20. Type or say in microphone "Switch on" then state of switch should be changed to on
21. Type or say in microphone "Switch off" then state of switch should be changed to off  
