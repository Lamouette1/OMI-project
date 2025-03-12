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
        
        <!-- Language selector -->
        <div class="language-selector">
          <button 
            @click="changeLanguage('en-US')" 
            class="language-button" 
            :class="{ 'active': currentLanguage === 'en-US' }">
            <i class="fa-solid fa-flag-usa"></i> English
          </button>
          <button 
            @click="changeLanguage('fr-FR')" 
            class="language-button" 
            :class="{ 'active': currentLanguage === 'fr-FR' }">
            <i class="fa-solid fa-flag"></i> Français
          </button>
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
              <p class="hint" v-if="currentLanguage === 'en-US'">
                Try saying <span>"start recording"</span> or <span>"how to work efficiently"</span>
              </p>
              <p class="hint" v-else>
                Essayez de dire <span>"commence l'enregistrement"</span> ou <span>"dit moi comment bien travailler"</span>
              </p>
            </div>
          </div>
        </div>
        
        <!-- Assistant Response -->
        <div v-if="assistantResponse" class="response-container">
          <div class="response-header">
            <i class="fa-solid fa-robot"></i>
            <span>{{ currentLanguage === 'en-US' ? 'Assistant Response' : 'Réponse de l\'assistant' }}</span>
            <button @click="clearAssistantResponse" class="icon-button close-button">
              <i class="fa-solid fa-times"></i>
            </button>
          </div>
          <div class="response-content">
            <p>{{ assistantResponse }}</p>
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
    
    <!-- Sport Calendar -->
    <div v-if="showCalendar" class="card calendar-card">
      <div class="card-header">
        <h2>{{ currentLanguage === 'en-US' ? 'Sports Calendar' : 'Calendrier Sportif' }}</h2>
        <button @click="showCalendar = false" class="icon-button">
          <i class="fa-solid fa-times"></i>
        </button>
      </div>
      <div class="calendar-content">
        <div v-for="(day, index) in getCurrentWeekdays()" :key="index" class="calendar-day">
          <div class="day-header">{{ day }}</div>
          <div class="day-content">
            <div class="sport-item">{{ getCurrentSportSchedule()[index] }}</div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="card commands-card">
      <div class="card-header">
        <h2>{{ currentLanguage === 'en-US' ? 'Voice Commands' : 'Commandes Vocales' }}</h2>
      </div>
      <div class="card-body">
        <ul class="commands-list" v-if="currentLanguage === 'en-US'">
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
          <li>
            <div class="command-tag">Show Calendar</div>
            <div class="command-description">Display the sports schedule for the week</div>
          </li>
          <li>
            <div class="command-tag">How to work efficiently</div>
            <div class="command-description">Get tips on working efficiently</div>
          </li>
        </ul>
        <ul class="commands-list" v-else>
          <li>
            <div class="command-tag">Commence l'enregistrement</div>
            <div class="command-description">Commencer à capturer l'audio depuis ESP32</div>
          </li>
          <li>
            <div class="command-tag">Arrête l'enregistrement</div>
            <div class="command-description">Terminer la session d'enregistrement actuelle</div>
          </li>
          <li>
            <div class="command-tag">Efface la transcription</div>
            <div class="command-description">Supprimer tout le texte de la transcription</div>
          </li>
          <li>
            <div class="command-tag">Montre le calendrier sportif</div>
            <div class="command-description">Afficher le calendrier sportif pour la semaine</div>
          </li>
          <li>
            <div class="command-tag">Dit moi comment bien travailler</div>
            <div class="command-description">Obtenir des conseils pour bien travailler</div>
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
      esp32Address: 'ws://your-esp32-ip:port',
      // Language settings
      currentLanguage: 'en-US',
      // Calendar data
      showCalendar: false,
      weekdays: {
        'en-US': ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'],
        'fr-FR': ['Lundi', 'Mardi', 'Mercredi', 'Jeudi', 'Vendredi', 'Samedi', 'Dimanche']
      },
      sportSchedule: {
        'en-US': ['Running', 'Swimming', 'Weight Training', 'Yoga', 'Tennis', 'Rest', 'Walking'],
        'fr-FR': ['Course à pied', 'Natation', 'Musculation', 'Yoga', 'Tennis', 'Repos', 'Marche']
      },
      // Assistant response
      assistantResponse: '',
      // Common questions and answers
      knowledgeBase: {
        'en-US': {
          'work efficiently': `To work efficiently, try these tips:
1. Break large tasks into smaller, manageable chunks
2. Use the Pomodoro technique: 25 minutes of focus followed by a 5-minute break
3. Prioritize tasks using the Eisenhower Matrix
4. Minimize distractions by turning off notifications
5. Set clear goals for each work session
6. Take regular breaks to maintain mental freshness
7. Stay hydrated and maintain proper posture
8. Use task management tools to keep track of your work`,
          
          'productivity': `To improve productivity:
1. Start your day with the most challenging tasks
2. Keep a to-do list and update it regularly
3. Automate repetitive tasks when possible
4. Learn keyboard shortcuts for frequently used applications
5. Use time-blocking in your calendar
6. Limit multitasking and focus on one task at a time
7. Get enough sleep and exercise regularly
8. Create a dedicated workspace with minimal distractions`
        },
        'fr-FR': {
          'bien travailler': `Pour bien travailler, essayez ces conseils :
1. Divisez les grandes tâches en petites parties gérables
2. Utilisez la technique Pomodoro : 25 minutes de concentration suivies d'une pause de 5 minutes
3. Priorisez vos tâches à l'aide de la matrice d'Eisenhower
4. Minimisez les distractions en désactivant les notifications
5. Fixez des objectifs clairs pour chaque session de travail
6. Prenez des pauses régulières pour garder l'esprit frais
7. Restez hydraté et maintenez une bonne posture
8. Utilisez des outils de gestion de tâches pour suivre votre travail`,
          
          'productivité': `Pour améliorer votre productivité :
1. Commencez votre journée par les tâches les plus difficiles
2. Tenez une liste de tâches et mettez-la à jour régulièrement
3. Automatisez les tâches répétitives lorsque c'est possible
4. Apprenez les raccourcis clavier pour les applications fréquemment utilisées
5. Utilisez le système de blocage de temps dans votre calendrier
6. Limitez le multitâche et concentrez-vous sur une tâche à la fois
7. Dormez suffisamment et faites régulièrement de l'exercice
8. Créez un espace de travail dédié avec un minimum de distractions`
        }
      }
    }
  },
  mounted() {
    this.setupSpeechRecognition();
    this.connectToESP32();
  },
  methods: {
    setupSpeechRecognition() {
      // Check if browser supports SpeechRecognition
      if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        this.recognition = new SpeechRecognition();
        this.recognition.continuous = true;
        this.recognition.interimResults = true;
        this.recognition.lang = this.currentLanguage;
        
        this.recognition.onresult = (event) => {
          let interimTranscript = '';
          let finalTranscript = '';
          
          for (let i = event.resultIndex; i < event.results.length; i++) {
            const transcript = event.results[i][0].transcript;
            if (event.results[i].isFinal) {
              finalTranscript += transcript;
              this.processVoiceCommand(transcript);
            } else {
              interimTranscript += transcript;
            }
          }
          
          this.transcript = finalTranscript || interimTranscript;
        };
        
        this.recognition.onerror = (event) => {
          console.error('Speech recognition error', event.error);
        };
      } else {
        alert('Your browser does not support speech recognition.');
      }
    },
    
    processVoiceCommand(transcript) {
      const lowerTranscript = transcript.toLowerCase().trim();
      
      if (this.currentLanguage === 'en-US') {
        // Process English commands
        if (lowerTranscript.includes('start recording')) {
          this.startRecording();
        } else if (lowerTranscript.includes('stop recording')) {
          this.stopRecording();
        } else if (lowerTranscript.includes('clear transcript')) {
          this.clearTranscript();
        } else if (
          lowerTranscript.includes('show calendar') || 
          lowerTranscript.includes('display calendar') ||
          lowerTranscript.includes('sports calendar')
        ) {
          this.showCalendar = true;
        } else if (
          lowerTranscript.includes('how to work efficiently') ||
          lowerTranscript.includes('work efficiently') ||
          lowerTranscript.includes('efficiently work')
        ) {
          this.generateAssistantResponse('work efficiently');
        } else if (
          lowerTranscript.includes('improve productivity') ||
          lowerTranscript.includes('productivity tips') ||
          lowerTranscript.includes('be more productive')
        ) {
          this.generateAssistantResponse('productivity');
        }
      } else if (this.currentLanguage === 'fr-FR') {
        // Process French commands
        if (
          lowerTranscript.includes('commence l\'enregistrement') || 
          lowerTranscript.includes('commencer l\'enregistrement') ||
          lowerTranscript.includes('démarrer l\'enregistrement')
        ) {
          this.startRecording();
        } else if (
          lowerTranscript.includes('arrête l\'enregistrement') || 
          lowerTranscript.includes('arrêter l\'enregistrement') ||
          lowerTranscript.includes('stopper l\'enregistrement')
        ) {
          this.stopRecording();
        } else if (
          lowerTranscript.includes('efface la transcription') ||
          lowerTranscript.includes('effacer la transcription') ||
          lowerTranscript.includes('nettoyer')
        ) {
          this.clearTranscript();
        } else if (
          lowerTranscript.includes('montre le calendrier') ||
          lowerTranscript.includes('afficher le calendrier') ||
          lowerTranscript.includes('calendrier sportif') ||
          lowerTranscript.includes('calendrier du sport') ||
          lowerTranscript.includes('calendrier pour le sport')
        ) {
          this.showCalendar = true;
        } else if (
          lowerTranscript.includes('comment bien travailler') ||
          lowerTranscript.includes('dit moi comment bien travailler') ||
          lowerTranscript.includes('conseils pour bien travailler')
        ) {
          this.generateAssistantResponse('bien travailler');
        } else if (
          lowerTranscript.includes('améliorer la productivité') ||
          lowerTranscript.includes('productivité') ||
          lowerTranscript.includes('être plus productif')
        ) {
          this.generateAssistantResponse('productivité');
        }
      }
    },
    
    generateAssistantResponse(topic) {
      const languageResponses = this.knowledgeBase[this.currentLanguage];
      
      if (languageResponses && languageResponses[topic]) {
        this.assistantResponse = languageResponses[topic];
      } else {
        // Fallback response if the topic isn't found
        this.assistantResponse = this.currentLanguage === 'en-US' 
          ? "I'm sorry, I don't have information on that topic yet."
          : "Désolé, je n'ai pas encore d'informations sur ce sujet.";
      }
    },
    
    clearAssistantResponse() {
      this.assistantResponse = '';
    },
    
    changeLanguage(lang) {
      if (this.currentLanguage !== lang) {
        this.currentLanguage = lang;
        
        if (this.recognition) {
          // Restart recognition with new language if it's currently active
          const wasListening = this.isListening;
          
          if (wasListening) {
            this.recognition.stop();
          }
          
          this.recognition.lang = lang;
          
          if (wasListening) {
            setTimeout(() => {
              this.recognition.start();
            }, 200);
          }
        }
        
        // Clear assistant response when changing language
        this.assistantResponse = '';
      }
    },
    
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
        // For demo purposes, simulate a connection:
        setTimeout(() => {
          this.esp32Connected = true;
          console.log('Connected to ESP32 (simulated)');
        }, 1500);
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
    },
    
    getCurrentWeekdays() {
      return this.weekdays[this.currentLanguage];
    },
    
    getCurrentSportSchedule() {
      return this.sportSchedule[this.currentLanguage];
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

/* Calendar styles */
.calendar-card {
  margin-top: 20px;
}

.calendar-content {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
}

.calendar-day {
  border-right: 1px solid #edf2f7;
  border-bottom: 1px solid #edf2f7;
}

.calendar-day:last-child {
  border-right: none;
}

.day-header {
  padding: 12px 8px;
  text-align: center;
  background-color: #f7fafc;
  font-weight: 600;
  border-bottom: 1px solid #edf2f7;
}

.day-content {
  min-height: 100px;
  padding: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.sport-item {
  background-color: #ebf8ff;
  color: #3182ce;
  padding: 10px;
  border-radius: 6px;
  text-align: center;
  font-weight: 500;
  width: 100%;
}

/* Language selector */
.language-selector {
  display: flex;
  align-items: center;
  gap: 10px;
}

.language-button {
  padding: 8px 12px;
  border: 1px solid #e2e8f0;
  border-radius: 20px;
  background: white;
  color: #718096;
  font-size: 0.85rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  gap: 5px;
}

.language-button i {
  font-size: 0.85rem;
}

.language-button:hover {
  background: #f7fafc;
}

.language-button.active {
  background: #4299e1;
  color: white;
  border-color: #4299e1;
}

/* Response container */
.response-container {
  margin-top: 20px;
  background: white;
  border-radius: 8px;
  overflow: hidden;
  border-left: 4px solid #4299e1;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
  animation: fadeIn 0.3s ease-in-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.response-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 15px;
  background: #ebf8ff;
  color: #2b6cb0;
  font-weight: 600;
}

.response-header i {
  font-size: 1.1rem;
}

.response-header .close-button {
  margin-left: auto;
  color: #2b6cb0;
}

.response-content {
  padding: 15px 20px;
}

.response-content p {
  margin: 0;
  line-height: 1.7;
  color: #2d3748;
  white-space: pre-line;
}

@media (max-width: 768px) {
  .card-header {
    flex-direction: column;
    gap: 15px;
  }
  
  .language-selector {
    width: 100%;
    justify-content: center;
  }
  
  .calendar-content {
    grid-template-columns: 1fr;
  }
  
  .calendar-day {
    border-right: none;
  }
}
</style>
