# Expense_Tracker
 an expense tracking application where users can log their expenses and view a breakdown of their spending by categories.  the app displays monthly summaries and allow users to set spending limits for each category (e.g., food, entertainment, bills).



## tools for project
Front-end: React.js for UI, Chart.js for visualizing expenses
back-end: firebase
Database: Firebase(NoSQL)
hosting: Firebase for front-end and back-end


## step1: planning the Architecture

Before diving into code, have a clear picture of what your application will look like and what it will do. Here's a basic breakdown:

User Authentication: Users must be able to sign up, log in, and log out.
Expense Management: Users can add, view, edit, and delete expenses. Each expense has fields like title, amount, date, and category.
Dashboard: A visual representation (using charts) of the user’s expenses over time (daily, monthly, yearly).
Backend API: A RESTful API for managing expenses, users, and data interactions.
Database: Stores user data and expenses.

## Step 2: Set Up the Development Environment
1.1. Install Node.js and npm
Go to nodejs.org to download and install Node.js. This will also install npm (Node Package Manager).
1.2. Install Expo CLI
Expo simplifies React Native development and allows live testing on mobile devices.

Run the following command in your terminal to install Expo globally:

bash
Copy code
npm install -g expo-cli
1.3. Create a New React Native Project
Create a new React Native project using Expo:

bash
Copy code
expo init ExpenseTrackerApp
Choose the Blank template.

Navigate into your project folder:

bash
Copy code
cd ExpenseTrackerApp
Start the development server:

bash
Copy code
expo start
Open the Expo Go app on your phone and scan the QR code to see the app running live.

## Step 3: Set Up Firebase
2.1. Create a Firebase Project
Go to Firebase Console.
Click Add Project and follow the instructions.
Enable the following services:
Authentication: Select the email/password method.
Firestore: Set up a Cloud Firestore database to store expenses.
2.2. Install Firebase SDK
In your project, install Firebase:

bash
Copy code
npm install firebase
2.3. Configure Firebase in Your React Native App
Create a firebase.js file in your project to initialize Firebase:

javascript
Copy code
// firebase.js
import firebase from 'firebase/app';
import 'firebase/auth';
import 'firebase/firestore';

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};

if (!firebase.apps.length) {
  firebase.initializeApp(firebaseConfig);
}

export const auth = firebase.auth();
export const db = firebase.firestore();
Replace the configuration values (apiKey, authDomain, etc.) with your Firebase project credentials from the Firebase Console.

## Step 4: Set Up Navigation
3.1. Install React Navigation
React Navigation is needed for navigating between screens (e.g., login, dashboard, expense input).

bash
Copy code
npm install @react-navigation/native
npm install @react-navigation/stack
expo install react-native-screens react-native-safe-area-context
3.2. Configure Navigation
Modify App.js to include basic navigation:

javascript
Copy code
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import SignUpScreen from './screens/SignUpScreen';
import LoginScreen from './screens/LoginScreen';
import DashboardScreen from './screens/DashboardScreen';

const Stack = createStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Login">
        <Stack.Screen name="Login" component={LoginScreen} />
        <Stack.Screen name="SignUp" component={SignUpScreen} />
        <Stack.Screen name="Dashboard" component={DashboardScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
## Step 5: Implement Authentication with Firebase
4.1. Create SignUpScreen.js
In the screens folder, create a file SignUpScreen.js:

javascript
Copy code
import React, { useState } from 'react';
import { View, TextInput, Button } from 'react-native';
import { auth } from '../firebase';

const SignUpScreen = ({ navigation }) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSignUp = () => {
    auth.createUserWithEmailAndPassword(email, password)
      .then(userCredentials => {
        console.log('Registered:', userCredentials.user.email);
        navigation.navigate('Dashboard');
      })
      .catch(error => alert(error.message));
  };

  return (
    <View>
      <TextInput placeholder="Email" value={email} onChangeText={setEmail} />
      <TextInput placeholder="Password" value={password} onChangeText={setPassword} secureTextEntry />
      <Button title="Sign Up" onPress={handleSignUp} />
    </View>
  );
};

export default SignUpScreen;
4.2. Create LoginScreen.js
Create a file LoginScreen.js:

javascript
Copy code
import React, { useState } from 'react';
import { View, TextInput, Button } from 'react-native';
import { auth } from '../firebase';

const LoginScreen = ({ navigation }) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = () => {
    auth.signInWithEmailAndPassword(email, password)
      .then(userCredentials => {
        console.log('Logged in:', userCredentials.user.email);
        navigation.navigate('Dashboard');
      })
      .catch(error => alert(error.message));
  };

  return (
    <View>
      <TextInput placeholder="Email" value={email} onChangeText={setEmail} />
      <TextInput placeholder="Password" value={password} onChangeText={setPassword} secureTextEntry />
      <Button title="Login" onPress={handleLogin} />
    </View>
  );
};

export default LoginScreen;
## Step 6: Set Up Firestore for Storing Expenses
5.1. Create DashboardScreen.js
Create a file DashboardScreen.js to allow users to add and view their expenses:

javascript
Copy code
import React, { useState, useEffect } from 'react';
import { View, TextInput, Button, FlatList, Text } from 'react-native';
import { db, auth } from '../firebase';

const DashboardScreen = () => {
  const [expense, setExpense] = useState('');
  const [amount, setAmount] = useState('');
  const [expenses, setExpenses] = useState([]);

  useEffect(() => {
    const fetchExpenses = () => {
      db.collection('expenses')
        .where('userId', '==', auth.currentUser.uid)
        .onSnapshot(snapshot => {
          const expensesData = snapshot.docs.map(doc => ({
            id: doc.id,
            ...doc.data()
          }));
          setExpenses(expensesData);
        });
    };
    fetchExpenses();
  }, []);

  const handleAddExpense = () => {
    db.collection('expenses').add({
      userId: auth.currentUser.uid,
      expense,
      amount,
      createdAt: firebase.firestore.FieldValue.serverTimestamp()
    });
    setExpense('');
    setAmount('');
  };

  return (
    <View>
      <TextInput placeholder="Expense Name" value={expense} onChangeText={setExpense} />
      <TextInput placeholder="Amount" value={amount} onChangeText={setAmount} keyboardType="numeric" />
      <Button title="Add Expense" onPress={handleAddExpense} />

      <FlatList
        data={expenses}
        renderItem={({ item }) => (
          <View>
            <Text>{item.expense}: ${item.amount}</Text>
          </View>
        )}
        keyExtractor={item => item.id}
      />
    </View>
  );
};

export default DashboardScreen;
## Step 7: Testing and Deployment
6.1. Test the App
Use Expo Go to scan the QR code and test the app on your device.
6.2. Build for Android and iOS
Run the following commands to build your app for Android or iOS:

bash
Copy code
expo build:android
expo build:ios


## Conclusion
You now have a clear step-by-step guide for building a mobile Expense Tracker using React Native (Expo) and Firebase. This stack simplifies authentication and data management, eliminating the need for a separate backend (like Flask). You can add additional features like real-time updates or notifications using Firebase Cloud Functions in the future.