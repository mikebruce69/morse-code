# morse-code
mport React, { useState, useEffect, useRef } from 'react';
import { 
  View, 
  Text, 
  StyleSheet, 
  TextInput, 
  TouchableOpacity, 
  ScrollView, 
  Pressable,
  Platform,
  FlatList 
} from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { Feather, MaterialCommunityIcons } from '@expo/vector-icons';
import { Audio } from 'expo-av';
import { StatusBar } from 'expo-status-bar';

// Morse code mappings
const morseCodeMap = {
  'A': '.-', 'B': '-...', 'C': '-.-.', 'D': '-..', 'E': '.', 'F': '..-.', 'G': '--.', 'H': '....', 'I': '..', 'J': '.---',
  'K': '-.-', 'L': '.-..', 'M': '--', 'N': '-.', 'O': '---', 'P': '.--.', 'Q': '--.-', 'R': '.-.', 'S': '...', 'T': '-',
  'U': '..-', 'V': '...-', 'W': '.--', 'X': '-..-', 'Y': '-.--', 'Z': '--..',
  '0': '-----', '1': '.----', '2': '..---', '3': '...--', '4': '....-', '5': '.....', '6': '-....', '7': '--...', '8': '---..', '9': '----.',
  '.': '.-.-.-', ',': '--..--', '?': '..--..', "'": '.----.', '!': '-.-.--', '/': '-..-.', '(': '-.--.', ')': '-.--.-',
  '&': '.-...', ':': '---...', ';': '-.-.-.', '=': '-...-', '+': '.-.-.', '-': '-....-', '_': '..--.-', '"': '.-..-.', '$': '...-..-', '@': '.--.-.',
  ' ': '/'
};

// Invert morse code map for decoding
const invertedMorseCodeMap = Object.entries(morseCodeMap).reduce((acc, [key, value]) => ({
  ...acc,
  [value]: key
}), {});

export default function HomeScreen() {
  const [text, setText] = useState('');
  const [morse, setMorse] = useState('');
  const [activeTab, setActiveTab] = useState('encode');
  const [messages, setMessages] = useState([]);
  const [name, setName] = useState('You');
  const [isPlaying, setIsPlaying] = useState(false);
  const [showVisualizer, setShowVisualizer] = useState(false);
  const soundObject = useRef(new Audio.Sound()).current;
  const scrollViewRef = useRef(null);

  const textToMorse = (text) => {
    return text
      .toUpperCase()
      .split('')
      .map(char => morseCodeMap[char] || char)
      .join(' ');
  };

  const morseToText = (morse) => {
    return morse
      .split(' ')
      .map(code => invertedMorseCodeMap[code] || code)
      .join('');
  };

  const handleTextChange = (newText) => {
    setText(newText);
    setMorse(textToMorse(newText));
  };

  const handleMorseChange = (newMorse) => {
    setMorse(newMorse);
    setText(morseToText(newMorse));
  };

  const playMorseCode = async () => {
    if (isPlaying) return;
    setIsPlaying(true);
    setShowVisualizer(true);
    
    try {
      for (let i = 0; i < morse.length; i++) {
        const char = morse[i];
        
        if (char === '.') {
          await playSound(100);
          await sleep(100);
        } else if (char === '-') {
          await playSound(300);
          await sleep(100);
        } else if (char === ' ') {
          await sleep(300);
        } else if (char === '/') {
          await sleep(700);
        }
      }
    } catch (error) {
      console.error("Failed to play morse code:", error);
    } finally {
      setIsPlaying(false);
      setShowVisualizer(false);
    }
  };

  const playSound = async (duration) => {
    try {
      await soundObject.unloadAsync();
      await soundObject.loadAsync(require('../assets/beep.json'));
      await soundObject.playAsync();
      return sleep(duration);
    } catch (error) {
      console.error("Error playing sound:", error);
    }
  };

  const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));

  const sendMessage = () => {
    if (!text.trim()) return;
    
    const newMessage = {
      id: Date.now().toString(),
      sender: name,
      text: text,
      morse: morse,
      timestamp: new Date().toLocaleTimeString()
    };
    
    setMessages([...messages, newMessage]);
    setText('');
    setMorse('');
    
    setTimeout(() => {
      scrollViewRef.current?.scrollToEnd({ animated: true });
    }, 100);
  };

  return (
    <SafeAreaView style={styles.container}>
      <StatusBar style="dark" />
      <View style={styles.header}>
        <Text style={styles.headerTitle}>Morse Code Communicator</Text>
      </View>
      
      <View style={styles.tabs}>
        <TouchableOpacity 
          style={[styles.tab, activeTab === 'encode' && styles.activeTab]} 
          onPress={() => setActiveTab('encode')}
        >
          <Text style={[styles.tabText, activeTab === 'encode' && styles.activeTabText]}>
            Text to Morse
          </Text>
        </TouchableOpacity>
        <TouchableOpacity 
          style={[styles.tab, activeTab === 'decode' && styles.activeTab]} 
          onPress={() => setActiveTab('decode')}
        >
          <Text style={[styles.tabText, activeTab === 'decode' && styles.activeTabText]}>
            Morse to Text
          </Text>
        </TouchableOpacity>
        <TouchableOpacity 
          style={[styles.tab, activeTab === 'chat' && styles.activeTab]} 
          onPress={() => setActiveTab('chat')}
        >
          <Text style={[styles.tabText, activeTab === 'chat' && styles.activeTabText]}>
            Chat
          </Text>
        </TouchableOpacity>
      </View>
      
      {activeTab === 'encode' && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Enter Text</Text>
          <TextInput
            style={styles.input}
            value={text}
            onChangeText={handleTextChange}
            placeholder="Enter text to convert to Morse code"
            multiline
          />
          
          <Text style={styles.sectionTitle}>Morse Code</Text>
          <View style={styles.morseContainer}>
            <Text style={styles.morseOutput}>{morse}</Text>
          </View>
          
          <TouchableOpacity 
            style={[styles.playButton, isPlaying && styles.disabledButton]} 
            onPress={playMorseCode}
            disabled={isPlaying || !morse}
          >
            <MaterialCommunityIcons name="play" size={20} color="white" />
            <Text style={styles.buttonText}>Play Morse Code</Text>
          </TouchableOpacity>
          
          {showVisualizer && (
            <View style={styles.visualizer}>
              {morse.split('').map((char, index) => (
                <View key={index} style={[
                  styles.visualizerItem,
                  char === '.' ? styles.dot : null,
                  char === '-' ? styles.dash : null,
                  char === ' ' ? styles.space : null,
                  char === '/' ? styles.wordSpace : null,
                ]} />
              ))}
            </View>
          )}
        </View>
      )}
      
      {activeTab === 'decode' && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Enter Morse Code</Text>
          <Text style={styles.helperText}>Use dots (.), dashes (-), spaces between letters, and forward slash (/) for word spacing</Text>
          <TextInput
            style={styles.input}
            value={morse}
            onChangeText={handleMorseChange}
            placeholder="Enter Morse code to convert to text"
            multiline
          />
          
          <Text style={styles.sectionTitle}>Decoded Text</Text>
          <View style={styles.morseContainer}>
            <Text style={styles.morseOutput}>{text}</Text>
          </View>
        </View>
      )}
      
      {activeTab === 'chat' && (
        <View style={styles.chatContainer}>
          <View style={styles.nameInputContainer}>
            <Text style={styles.sectionTitle}>Your Name:</Text>
            <TextInput
              style={styles.nameInput}
              value={name}
              onChangeText={setName}
              placeholder="Enter your name"
            />
          </View>
          
          <View style={styles.messagesContainer}>
            <FlatList
              ref={scrollViewRef}
              data={messages}
              keyExtractor={(item) => item.id}
              renderItem={({item}) => (
                <View style={styles.messageItem}>
                  <View style={styles.messageHeader}>
                    <Text style={styles.messageSender}>{item.sender}</Text>
                    <Text style={styles.messageTime}>{item.timestamp}</Text>
                  </View>
                  <Text style={styles.messageText}>{item.text}</Text>
                  <Text style={styles.messageMorse}>{item.morse}</Text>
                </View>
              )}
              contentContainerStyle={styles.messagesList}
            />
          </View>
          
          <View style={styles.inputContainer}>
            {activeTab === 'chat' && (
              <>
                <TextInput
                  style={styles.chatInput}
                  value={text}
                  onChangeText={handleTextChange}
                  placeholder="Type a message..."
                />
                <Text style={styles.morsePreview}>{morse}</Text>
                <TouchableOpacity style={styles.sendButton} onPress={sendMessage}>
                  <Feather name="send" size={20} color="white" />
                </TouchableOpacity>
              </>
            )}
          </View>
        </View>
      )}
      
      <View style={styles.footer}>
        <Text style={styles.footerText}>
          Made with ❤️ - Learn Morse Code to communicate!
        </Text>
      </View>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f7',
  },
  header: {
    padding: 16,
    backgroundColor: '#ffffff',
    borderBottomWidth: 1,
    borderBottomColor: '#e1e1e1',
    alignItems: 'center',
  },
  headerTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
  },
  tabs: {
    flexDirection: 'row',
    backgroundColor: '#ffffff',
    borderBottomWidth: 1,
    borderBottomColor: '#e1e1e1',
  },
  tab: {
    flex: 1,
    paddingVertical: 12,
    alignItems: 'center',
  },
  activeTab: {
    borderBottomWidth: 2,
    borderBottomColor: '#0066cc',
  },
  tabText: {
    color: '#666',
    fontWeight: '500',
  },
  activeTabText: {
    color: '#0066cc',
    fontWeight: 'bold',
  },
  section: {
    padding: 16,
    backgroundColor: '#ffffff',
    margin: 10,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 2,
    flex: 1,
  },
  sectionTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
    color: '#333',
  },
  helperText: {
    fontSize: 12,
    color: '#666',
    marginBottom: 8,
    fontStyle: 'italic',
  },
  input: {
    backgroundColor: '#f9f9f9',
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    fontSize: 16,
    minHeight: 80,
    marginBottom: 16,
  },
  morseContainer: {
    backgroundColor: '#f9f9f9',
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    minHeight: 80,
    marginBottom: 16,
  },
  morseOutput: {
    fontSize: 16,
    color: '#333',
    lineHeight: 24,
  },
  playButton: {
    backgroundColor: '#0066cc',
    borderRadius: 8,
    paddingVertical: 12,
    paddingHorizontal: 16,
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    marginTop: 8,
  },
  disabledButton: {
    backgroundColor: '#999',
  },
  buttonText: {
    color: 'white',
    fontWeight: 'bold',
    marginLeft: 8,
  },
  visualizer: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    marginTop: 20,
    padding: 10,
    backgroundColor: '#e0e0e0',
    borderRadius: 8,
    alignItems: 'center',
  },
  visualizerItem: {
    margin: 3,
  },
  dot: {
    width: 15,
    height: 15,
    backgroundColor: '#0066cc',
    borderRadius: 7.5,
  },
  dash: {
    width: 30,
    height: 10,
    backgroundColor: '#0066cc',
    borderRadius: 5,
  },
  space: {
    width: 15,
    height: 10,
  },
  wordSpace: {
    width: 30,
    height: 10,
  },
  chatContainer: {
    flex: 1,
    padding: 10,
  },
  nameInputContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 10,
    backgroundColor: '#ffffff',
    padding: 10,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 2,
    elevation: 1,
  },
  nameInput: {
    flex: 1,
    backgroundColor: '#f9f9f9',
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 8,
    marginLeft: 10,
    fontSize: 14,
  },
  messagesContainer: {
    flex: 1,
    backgroundColor: '#ffffff',
    borderRadius: 8,
    padding: 10,
    marginBottom: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 2,
    elevation: 1,
  },
  messagesList: {
    paddingBottom: 10,
  },
  messageItem: {
    backgroundColor: '#f0f7ff',
    padding: 12,
    borderRadius: 8,
    marginVertical: 5,
  },
  messageHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 4,
  },
  messageSender: {
    fontWeight: 'bold',
    color: '#0066cc',
  },
  messageTime: {
    fontSize: 12,
    color: '#666',
  },
  messageText: {
    fontSize: 15,
    marginBottom: 4,
  },
  messageMorse: {
    fontSize: 13,
    color: '#666',
    fontFamily: Platform.OS === 'ios' ? 'Courier' : 'monospace',
  },
  inputContainer: {
    backgroundColor: '#ffffff',
    borderRadius: 8,
    padding: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 2,
    elevation: 1,
  },
  chatInput: {
    backgroundColor: '#f9f9f9',
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 10,
    fontSize: 15,
    maxHeight: 100,
  },
  morsePreview: {
    fontSize: 12,
    color: '#666',
    marginVertical: 5,
    fontFamily: Platform.OS === 'ios' ? 'Courier' : 'monospace',
  },
  sendButton: {
    backgroundColor: '#0066cc',
    borderRadius: 8,
    padding: 10,
    alignItems: 'center',
    justifyContent: 'center',
  },
  footer: {
    padding: 16,
    alignItems: 'center',
  },
  footerText: {
    fontSize: 12,
    color: '#666',
  },
