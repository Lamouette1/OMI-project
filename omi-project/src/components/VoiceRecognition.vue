<template>
    <div class="voice-container">
      <div class="card main-card">
        <div class="card-header">
          <div class="status-badge" :class="{ 'listening': isListening }">
            <div class="status-icon">
              <div v-if="isListening" class="microphone-waves">
                <span></span>
                <span></span>
                <span></span>
                <span></span>
              </div>
              <i class="fa-solid" :class="isListening ? 'fa-microphone' : 'fa-microphone-slash'"></i>
            </div>
            <span>{{ isListening ? 'Listening' : 'Microphone Off' }}</span>
          </div>
        </div>
  
        <div class="card-body">
          <div class="transcript-container">
            <div class="transcript-header">
              <h2>Voice Transcript</h2>
              <button @click="clearTranscript" class="icon-button clear-button">
                <i class="fa-solid fa-eraser"></i>
              </button>
            </div>
            
            <div class="transcript-content" :class="{ 'empty': !transcript }">
              <div v-if="transcript" class="transcript-text">{{ transcript }}</div>
              <div v-else class="placeholder-text">
                <i class="fa-solid fa-comment-dots"></i>
                <p>Your spoken words will appear here...</p>
                <p class="hint">Try saying <span>"start recording"</span> or <span>"stop recording"</span></p>
              </div>
            </div>
          </div>
          
          <div v-if="isRecording" class="recording-indicator">
            <div class="recording-pulse"></div>
            <span>Recording in progress</span>
          </div>
        </div>
        
        <div class="card-footer">
          <button 
            @click="toggleListening" 
            class="control-button primary"
            :class="{ 'listening': isListening }">
            <i class="fa-solid" :class="isListening ? 'fa-stop-circle' : 'fa-play-circle'"></i>
            {{ isListening ? 'Stop Listening' : 'Start Listening' }}
          </button>
          
          <div class="device-status">
            <div class="status-dot" :class="{ 'connected': esp32Connected }"></div>
            <span>ESP32: {{ esp32Connected ? 'Connected' : 'Disconnected' }}</span>
          </div>
        </div>
      </div>
      
      <div class="card commands-card">
        <div class="card-header">
          <h2>Voice Commands</h2>
        </div>
        <div class="card-body">
          <ul class="commands-list">
            <li>
              <div class="command-tag">Start Recording</div>
              <div class="command-description">Begin capturing audio from ESP32</div>
            </li>
            <li>
              <div class="command-tag">Stop Recording</div>
              <div class="command-description">End the current recording session</div>
            </li>
            <li>
              <div class="command-tag">Clear Transcript</div>
              <div class="command-description">Remove all text from transcript</div>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        transcript: '',
        isListening: false,
        isRecording: false,
        recognition: null,
        socket: null,
        esp32Connected: false,
        esp32Address: 'ws://your-esp32-ip:port' // Change for ESP32's IP and port
      }
    },
    mounted() {
      // Check if browser supports SpeechRecognition
      if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        this.recognition = new SpeechRecognition();
        this.recognition.continuous = true;
        this.recognition.interimResults = true;
        
        this.recognition.onresult = (event) => {
          let interimTranscript = '';
          let finalTranscript = '';
          
          for (let i = event.resultIndex; i < event.results.length; i++) {
            const transcript = event.results[i][0].transcript;
            if (event.results[i].isFinal) {
              finalTranscript += transcript;
              
              // Check for commands
              const lowerTranscript = transcript.toLowerCase().trim();
              if (lowerTranscript.includes('start recording')) {
                this.startRecording();
              } else if (lowerTranscript.includes('stop recording')) {
                this.stopRecording();
              } else if (lowerTranscript.includes('clear transcript')) {
                this.clearTranscript();
              }
            } else {
              interimTranscript += transcript;
            }
          }
          
          this.transcript = finalTranscript || interimTranscript;
        };
        
        this.recognition.onerror = (event) => {
          console.error('Speech recognition error', event.error);
        };
        
        // Try to connect to ESP32
        this.connectToESP32();
      } else {
        alert('Your browser does not support speech recognition.');
      }
    },
    methods: {
      toggleListening() {
        if (this.isListening) {
          this.recognition.stop();
          this.isListening = false;
        } else {
          this.recognition.start();
          this.isListening = true;
        }
      },
      clearTranscript() {
        this.transcript = '';
      },
      startRecording() {
        this.isRecording = true;
        this.sendCommandToESP32('startRecording');
      },
      stopRecording() {
        this.isRecording = false;
        this.sendCommandToESP32('stopRecording');
      },
      connectToESP32() {
        try {
          // Comment this out if you're not actually connecting to an ESP32 yet
          // this.socket = new WebSocket(this.esp32Address);
          
          // For demo purposes, simulate a connection:
          setTimeout(() => {
            this.esp32Connected = true;
            console.log('Connected to ESP32 (simulated)');
          }, 1500);
          
          // Uncomment these when you're ready to connect to a real ESP32
          /*
          this.socket.onopen = () => {
            this.esp32Connected = true;
            console.log('Connected to ESP32');
          };
          
          this.socket.onmessage = (event) => {
            console.log('Message from ESP32:', event.data);
          };
          
          this.socket.onclose = () => {
            this.esp32Connected = false;
            console.log('Disconnected from ESP32');
            setTimeout(() => this.connectToESP32(), 3000);
          };
          
          this.socket.onerror = (error) => {
            console.error('WebSocket Error:', error);
          };
          */
        } catch (error) {
          console.error('Failed to connect to ESP32:', error);
        }
      },
      sendCommandToESP32(command) {
        if (this.socket && this.socket.readyState === WebSocket.OPEN) {
          this.socket.send(JSON.stringify({ command }));
        } else {
          console.log(`Command "${command}" would be sent to ESP32 (simulated)`);
        }
      }
    },
    beforeUnmount() {
      if (this.recognition) {
        this.recognition.stop();
      }
      if (this.socket) {
        this.socket.close();
      }
    }
  }
  </script>
  
  <style scoped>
  .voice-container {
    display: flex;
    flex-direction: column;
    gap: 20px;
    max-width: 900px;
    margin: 0 auto;
  }
  
  .card {
    background: white;
    border-radius: 12px;
    box-shadow: 0 8px 30px rgba(0, 0, 0, 0.06);
    overflow: hidden;
    transition: all 0.3s;
  }
  
  .main-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
  }
  
  .card-header {
    padding: 20px;
    background: #f9fafc;
    border-bottom: 1px solid #edf2f7;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .card-header h2 {
    margin: 0;
    font-weight: 600;
    color: #2d3748;
    font-size: 1.25rem;
  }
  
  .card-body {
    padding: 25px;
  }
  
  .card-footer {
    padding: 20px;
    background: #f9fafc;
    border-top: 1px solid #edf2f7;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .status-badge {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 8px 16px;
    border-radius: 30px;
    background: #f7fafc;
    color: #718096;
    font-weight: 500;
    transition: all 0.3s;
  }
  
  .status-badge.listening {
    background: #ebf8ff;
    color: #3182ce;
  }
  
  .status-icon {
    position: relative;
    width: 24px;
    height: 24px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .status-icon i {
    font-size: 16px;
  }
  
  .microphone-waves {
    position: absolute;
    width: 100%;
    height: 100%;
  }
  
  .microphone-waves span {
    position: absolute;
    width: 100%;
    height: 100%;
    border: 1px solid #3182ce;
    border-radius: 50%;
    animation: wave-animation 2s infinite;
    opacity: 0;
  }
  
  .microphone-waves span:nth-child(1) {
    animation-delay: 0s;
  }
  .microphone-waves span:nth-child(2) {
    animation-delay: 0.5s;
  }
  .microphone-waves span:nth-child(3) {
    animation-delay: 1s;
  }
  .microphone-waves span:nth-child(4) {
    animation-delay: 1.5s;
  }
  
  @keyframes wave-animation {
    0% {
      transform: scale(1);
      opacity: 0.8;
    }
    100% {
      transform: scale(2);
      opacity: 0;
    }
  }
  
  .transcript-container {
    background: #f9fafc;
    border-radius: 8px;
    overflow: hidden;
  }
  
  .transcript-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px 20px;
    background: #edf2f7;
  }
  
  .transcript-header h2 {
    margin: 0;
    font-size: 1rem;
    font-weight: 600;
    color: #4a5568;
  }
  
  .icon-button {
    background: none;
    border: none;
    color: #718096;
    font-size: 14px;
    cursor: pointer;
    padding: 5px;
    border-radius: 5px;
    transition: all 0.2s;
  }
  
  .icon-button:hover {
    background: rgba(0, 0, 0, 0.05);
    color: #2d3748;
  }
  
  .clear-button:hover {
    color: #e53e3e;
  }
  
  .transcript-content {
    min-height: 200px;
    max-height: 400px;
    overflow-y: auto;
    padding: 20px;
  }
  
  .transcript-text {
    white-space: pre-wrap;
    line-height: 1.7;
    color: #2d3748;
  }
  
  .placeholder-text {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100%;
    min-height: 200px;
    color: #a0aec0;
    text-align: center;
  }
  
  .placeholder-text i {
    font-size: 3rem;
    margin-bottom: 15px;
    opacity: 0.5;
  }
  
  .placeholder-text p {
    margin: 5px 0;
  }
  
  .hint {
    font-size: 0.85rem;
    margin-top: 10px;
  }
  
  .hint span {
    background: #ebf8ff;
    color: #3182ce;
    padding: 2px 6px;
    border-radius: 4px;
    font-weight: 500;
  }
  
  .control-button {
    padding: 12px 24px;
    border: none;
    border-radius: 8px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
    display: flex;
    align-items: center;
    gap: 8px;
  }
  
  .control-button i {
    font-size: 1.1rem;
  }
  
  .control-button.primary {
    background: #4299e1;
    color: white;
  }
  
  .control-button.primary:hover {
    background: #3182ce;
    box-shadow: 0 4px 12px rgba(66, 153, 225, 0.3);
  }
  
  .control-button.primary.listening {
    background: #e53e3e;
  }
  
  .control-button.primary.listening:hover {
    background: #c53030;
    box-shadow: 0 4px 12px rgba(229, 62, 62, 0.3);
  }
  
  .device-status {
    display: flex;
    align-items: center;
    gap: 8px;
    color: #718096;
    font-size: 0.85rem;
  }
  
  .status-dot {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background: #e53e3e;
    position: relative;
  }
  
  .status-dot.connected {
    background: #38a169;
  }
  
  .status-dot.connected::after {
    content: "";
    position: absolute;
    top: -2px;
    left: -2px;
    width: 14px;
    height: 14px;
    border-radius: 50%;
    background: rgba(56, 161, 105, 0.3);
    animation: pulse-green 2s infinite;
  }
  
  @keyframes pulse-green {
    0% {
      transform: scale(0.8);
      opacity: 0.8;
    }
    70% {
      transform: scale(1.3);
      opacity: 0;
    }
    100% {
      transform: scale(0.8);
      opacity: 0;
    }
  }
  
  .recording-indicator {
    margin-top: 20px;
    padding: 12px 16px;
    background: #fed7d7;
    border-left: 4px solid #e53e3e;
    border-radius: 4px;
    display: flex;
    align-items: center;
    gap: 12px;
    color: #c53030;
    font-weight: 600;
  }
  
  .recording-pulse {
    width: 14px;
    height: 14px;
    background: #e53e3e;
    border-radius: 50%;
    animation: recording-pulse 1.5s infinite;
  }
  
  @keyframes recording-pulse {
    0% {
      transform: scale(0.8);
      opacity: 0.8;
    }
    50% {
      transform: scale(1.2);
      opacity: 1;
    }
    100% {
      transform: scale(0.8);
      opacity: 0.8;
    }
  }
  
  .commands-card {
    background: #2d3748;
    color: white;
  }
  
  .commands-card .card-header {
    background: rgba(0, 0, 0, 0.2);
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  }
  
  .commands-card .card-header h2 {
    color: white;
  }
  
  .commands-list {
    list-style-type: none;
    padding: 0;
    margin: 0;
  }
  
  .commands-list li {
    padding: 15px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  }
  
  .commands-list li:last-child {
    border-bottom: none;
  }
  
  .command-tag {
    display: inline-block;
    padding: 4px 10px;
    background: #4299e1;
    color: white;
    border-radius: 4px;
    font-size: 0.9rem;
    font-weight: 600;
    margin-bottom: 8px;
  }
  
  .command-description {
    color: #a0aec0;
    font-size: 0.9rem;
  }
  </style>
  