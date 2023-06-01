# chat_app

![image](https://github.com/zaka1200/chat_app/assets/121964432/8a259197-84ec-41a4-926d-442913aa5da8)

#  table des matières:

- [Introduction](#introduction)
- [Serveur](#serveur)
- [Client](#client)
- [Threads](#Threads)
- [Swing](#Swing)
- [Sockets](#Sockets)
- [Test](#Test)
- [Conclusion](#conclusion)

# Introduction

Ce rapport examine deux codes qui constituent une application de chat en Java. Les codes, ChatServer et ChatClient, mettent en œuvre un système de chat basé sur des sockets TCP et une interface graphique Swing. Le ChatServer est responsable de la gestion des connexions des clients, de la diffusion des messages et de la gestion des déconnexions. Le ChatClient permet aux utilisateurs de se connecter au serveur, d'envoyer des messages et de recevoir des messages des autres utilisateurs connectés. Nous examinerons en détail chaque code, en expliquant les concepts clés tels que les threads, Swing et les sockets, et nous discuterons également du lien entre les deux codes.


# Serveur

#

Le code `ChatServer` représente le serveur de l'application de chat. Il utilise une `ServerSocket` pour accepter les connexions des clients. Chaque fois qu'un nouveau client se connecte, le serveur crée un nouveau thread `ClientHandler` pour gérer les communications avec ce client. Le `ClientHandler` lit les messages du client, les diffuse à tous les autres clients connectés et gère également les déconnexions des clients.

Le serveur utilise une liste pour conserver une référence à tous les `ClientHandler` actifs, ce qui lui permet de diffuser les messages à tous les clients. L'utilisation de threads distincts pour chaque client permet au serveur de gérer plusieurs connexions simultanément sans bloquer d'autres clients.

#
```java
import java.io.*;
import java.net.*;
import java.util.*;

public class ChatServer {
    private ServerSocket serverSocket;
    private List<ClientHandler> clients;

    public ChatServer(int port) {
        try {
            serverSocket = new ServerSocket(port);
            clients = new ArrayList<>();
            System.out.println("Serveur de chat démarré sur le port " + port);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
La classe `ChatServer` représente le côté serveur d'une application de chat. Elle a deux variables de classe : `serverSocket`, de type `ServerSocket`, utilisée pour écouter les connexions clients entrantes, et `clients`, de type `List<ClientHandler>`, qui contient les clients connectés au serveur.

Le constructeur `ChatServer` crée une instance de `ServerSocket` sur le port spécifié et initialise la liste des clients. S'il y a une exception lors de l'initialisation, elle est affichée.

```java
public void start() {
    while (true) {
        try {
            Socket socket = serverSocket.accept();
            System.out.println("Nouveau client connecté : " + socket);

            ClientHandler clientHandler = new ClientHandler(socket, this);
            clients.add(clientHandler);
            clientHandler.start();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
La méthode `start` est responsable d'accepter les connexions clients et de créer un nouveau thread `ClientHandler` pour chaque client. Elle s'exécute dans une boucle infinie et utilise `serverSocket.accept()` pour accepter les connexions clients entrantes. Lorsqu'un nouveau client se connecte, un objet `ClientHandler` est créé pour ce client, ajouté à la liste des clients et démarré en tant que thread.

```java
public void broadcastMessage(String message, ClientHandler sender) {
    for (ClientHandler client : clients) {
        if (client != sender) {
            client.sendMessage(sender.getUsername() + ": " + message);
        }
    }
}
```
La méthode `broadcastMessage` envoie un message à tous les clients connectés, sauf à l'expéditeur. Elle itère sur la liste des clients et utilise la méthode `sendMessage` pour envoyer le message à chaque client, en ajoutant le nom d'utilisateur de l'expéditeur au message.

```java
public void removeClient(ClientHandler client) {
    clients.remove(client);
}
```
La méthode `removeClient` supprime un client de la liste des clients lorsque celui-ci se déconnecte.

```java
public static void main(String[] args) {
    int port = 8080;
    ChatServer server = new ChatServer(port);
    server.start();
}
```
La méthode `main` crée une instance de `ChatServer` avec un port spécifié et démarre le serveur en appelant la méthode `start`.

```java
class ClientHandler extends Thread {
    private Socket socket;
    private ChatServer server;
    private PrintWriter writer;
    private String username;

    public ClientHandler(Socket socket, ChatServer server) {
        this.socket = socket;
        this.server = server;
    }
```
La classe `ClientHandler` représente un thread qui gère la communication avec un client spécifique. Elle a plusieurs variables d'instance : `socket`, de type `Socket`, représente la connexion socket avec le client ; `server`, de type `ChatServer`, est une référence au serve

ur auquel le client est connecté ; `writer`, de type `PrintWriter`, est utilisé pour envoyer des messages au client ; `username`, de type `String`, représente le nom d'utilisateur du client.

Le constructeur `ClientHandler` initialise les variables d'instance avec les valeurs fournies.

```java
public String getUsername() {
    return username;
}

public void sendMessage(String message) {
    writer.println(message);
}
```
La méthode `getUsername` renvoie le nom d'utilisateur du client.

La méthode `sendMessage` envoie un message au client en utilisant le `PrintWriter`.

```java
public void run() {
    try {
        InputStream input = socket.getInputStream();
        BufferedReader reader = new BufferedReader(new InputStreamReader(input));
        OutputStream output = socket.getOutputStream();
        writer = new PrintWriter(output, true);
        writer.println("Welcome to the group chat");
        username = reader.readLine();
        server.broadcastMessage(username + " a rejoint le chat.", this);
        String clientMessage;
        while ((clientMessage = reader.readLine()) != null) {
            server.broadcastMessage(clientMessage, this);
        }
        socket.close();
        server.removeClient(this);
        server.broadcastMessage(username + " a quitté le chat !!", this);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
La méthode `run` est exécutée lorsque le thread `ClientHandler` démarre. Elle gère la communication avec le client. Elle utilise `socket.getInputStream()` pour obtenir le flux d'entrée du client et `socket.getOutputStream()` pour obtenir le flux de sortie.

La première étape consiste à effectuer l'authentification du client en demandant son nom d'utilisateur et en le lisant à partir du flux d'entrée. Ensuite, le serveur envoie un message de bienvenue à tous les clients connectés en utilisant la méthode `broadcastMessage`.

Ensuite, la méthode entre dans une boucle où elle lit les messages envoyés par le client à partir du flux d'entrée et les diffuse à tous les autres clients en utilisant la méthode `broadcastMessage`.

Lorsque le client se déconnecte, la méthode ferme la connexion socket, supprime le client de la liste des clients et envoie un message à tous les clients pour informer qu'il a quitté le chat.

En cas d'exception lors de la communication, l'erreur est affichée.

# Client 

#

Le code `ChatClient` représente le client de l'application de chat. Il utilise un `Socket` pour se connecter au serveur. Une fois connecté, le client peut envoyer des messages au serveur et recevoir des messages des autres utilisateurs connectés. Le client utilise également l'interface graphique Swing pour créer une fenêtre de chat conviviale. Il utilise des composants tels que `JTextArea`, `JTextField` et `JButton` pour afficher les messages, saisir les messages et envoyer les messages respectivement.

Lorsqu'un utilisateur envoie un message, le client l'envoie au serveur via le `Socket` et l'affiche dans la zone de chat en utilisant `JTextArea`. Le client utilise également un mécanisme de couleur pour afficher les messages des utilisateurs dans des couleurs différentes, en utilisant `Color` de Swing.

#


1. Importations :
   ```java
   import java.io.*;
   import java.net.*;
   import javax.swing.*;
   import java.awt.*;
   import java.awt.event.*;
   ```

   Les instructions d'importation incluent les bibliothèques nécessaires pour travailler avec les entrées/sorties, les sockets, les composants Swing et les événements AWT.

2. Déclaration de classe et variables membres :
   ```java
   public class ChatClient {
       private Socket socket;
       private BufferedReader reader;
       private PrintWriter writer;
       private JTextArea chatArea;
       private JTextField messageField;
       private String username;
       
   }
   ```

   La classe `ChatClient` contient des variables membres pour stocker le socket, le lecteur (`reader`), l'écrivain (`writer`), la zone de chat (`chatArea`), le champ de message (`messageField`) et le nom d'utilisateur (`username`).

3. Constructeur `ChatClient` :
   ```java
   public ChatClient(String serverAddress, int serverPort) {
       try {
           socket = new Socket(serverAddress, serverPort);
           reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
           writer = new PrintWriter(socket.getOutputStream(), true);
           performAuthentication();
           createAndShowGUI();
           startReadingMessages();
       } catch (IOException e) {
           e.printStackTrace();
       }
   }
   ```

   Le constructeur `ChatClient` prend en paramètres l'adresse du serveur et le port du serveur. Il initialise le socket en se connectant au serveur, crée les flux d'entrée et de sortie pour la communication avec le serveur, effectue l'authentification avec le serveur, crée l'interface utilisateur et démarre la lecture des messages à partir du serveur.

4. Méthode `createAndShowGUI` :
   ```java
   public void createAndShowGUI() {
       try {
           UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
       } catch (Exception ex) {
           ex.printStackTrace();
       }
       chatArea = new JTextArea(20, 40);
       chatArea.setEditable(false);
       messageField = new JTextField(40);
       JButton sendButton = new JButton("Send");
       
   }
   ```

   La méthode `createAndShowGUI` crée l'interface utilisateur Swing pour la fenêtre de chat. Elle configure l'apparence en utilisant le look and feel système, crée les composants tels que `JTextArea`, `JTextField` et `JButton`, et organise ces composants dans des panneaux pour obtenir la mise en page souhaitée. Enfin, elle crée et affiche la fenêtre de chat.

5. Méthode `sendMessage` :
   ```java
   private void sendMessage(String message) {
       writer.println(message);
   }
   ```

   La méthode `sendMessage` envoie un message au serveur en l'écrivant dans le flux de sortie du socket.

6. Méthode `appendMessage` :
   ```java
   private void appendMessage(String message) {
       SwingUtilities.invokeLater(() -> chatArea.append(message + "\n"));
   }
   ```

   La méthode `appendMessage` ajoute un message à la zone de chat en utilisant `SwingUtilities.invokeLater` pour garantir que l'opération est effectuée sur le thread de l'interface utilisateur.

7. Méthode `performAuthentication` :
   ```java
   private void performAuthentication() throws IOException {
       username  = JOptionPane.showInputDialog(null, "Enter your username:");
       writer.println(username);
   }
   ```

   La méthode `performAuthentication` demande à l'utilisateur d'entrer son nom d'utilisateur en utilisant une boîte de dialogue Swing, puis envoie le nom d'utilisateur au serveur via le flux de sortie du socket.

8. Méthode `startReadingMessages` :
   ```java
   private void startReadingMessages() {
       Thread readThread = new Thread(() -> {
           try {
               String message;
               while ((message = reader.readLine()) != null) {
                   appendMessage(message);
               }
           } catch (IOException e) {
               e.printStackTrace();			
           }
       });
       readThread.start();
   }
   ```

   La méthode `startReadingMessages` crée un nouveau thread qui lit en boucle les messages provenant du serveur via le flux d'entrée du socket. Chaque message lu est ajouté à la zone de chat en utilisant la méthode `appendMessage`.

9. Méthode `main` :
   ```java
   public static void main(String[] args) {
       String serverAddress = "localhost";
       int serverPort = 8080;
       SwingUtilities.invokeLater(() -> new ChatClient(serverAddress, serverPort));
   }
   ```

   La méthode `main` est la méthode principale qui est exécutée lorsque le programme est lancé. Elle crée une instance de `ChatClient` en utilisant l'adresse du serveur et le port spécifiés. L'exécution est effectuée sur le thread de l'interface utilisateur en utilisant `SwingUtilities.invokeLater`.
# Threads 

![image](https://github.com/zaka1200/chat_app/assets/121964432/ff0a8f78-c1a8-4288-977f-f418f331c2e9)


Les threads jouent un rôle crucial dans ces deux codes. Dans le `ChatServer`, chaque connexion client est gérée par un thread `ClientHandler` distinct. Cela permet au serveur de gérer plusieurs connexions simultanément et de répondre aux messages de chaque client sans bloquer les autres clients. De même, dans le `ChatClient`, un thread distinct est utilisé pour lire les messages du serveur en continu tout en maintenant l'interface utilisateur réactive.

# Swing 

![image](https://github.com/zaka1200/chat_app/assets/121964432/c96bba1e-c927-45fa-9211-639b30e18ffd) ![image](https://github.com/zaka1200/chat_app/assets/121964432/e4fb2558-8a83-491e-b434-47c58805d459)


Swing est une bibliothèque graphique en Java qui permet de créer des interfaces graphiques. Dans le `ChatClient`, Swing est utilisé pour créer une fenêtre de chat conviviale avec des composants tels que `JTextArea`, `JTextField` et `JButton`. Swing fournit également des fonctionnalités pour la gestion des événements, telles que l'ajout d'un écouteur d'action au bouton "Envoyer". L'utilisation de Swing facilite la création d'une interface utilisateur interactive et réactive pour le client.

# Sockets 

![image](https://github.com/zaka1200/chat_app/assets/121964432/de8a8f7d-c65c-42bc-a041-370839481f64)


Les sockets sont utilisés pour établir une communication réseau entre le serveur et les clients. Le serveur utilise une `ServerSocket` pour écouter les connexions entrantes des clients, tandis que chaque client utilise un `Socket` pour se connecter au serveur. Les sockets permettent aux données d'être envoyées et reçues via le réseau, facilitant ainsi la communication bidirectionnelle entre le serveur et les clients.

# Test 

tout d abord on commence par compiler le code du serveur en utilisons javac 

![image](https://github.com/zaka1200/chat_app/assets/121964432/fa713f82-8f2e-45c3-8163-92b3822d939f)
 
 puis on lance le serveur 

![image](https://github.com/zaka1200/chat_app/assets/121964432/0d43787c-873f-4e01-b14b-db6af564509d)
 
maintenant a partir de deux terminal on lance 2 client :

![image](https://github.com/zaka1200/chat_app/assets/121964432/55092369-b1aa-4615-89a0-49f430f93472)

on s'authentifie :

![image](https://github.com/zaka1200/chat_app/assets/121964432/5e2a2b6e-4da9-405f-a12a-2498f21b99e1) ![image](https://github.com/zaka1200/chat_app/assets/121964432/058f5a6c-7d70-40f8-bbd8-04ff5896bb72)

on teste le chat :

![image](https://github.com/zaka1200/chat_app/assets/121964432/dd5655c2-edd6-4312-bb49-760a0a4f0adb)



# Consclusion 


En conclusion, les codes `ChatServer` et `ChatClient` présentent une implémentation basique d'une application de chat en Java. Le `ChatServer` gère les connexions des clients, la diffusion des messages et les déconnexions, tandis que le `ChatClient` permet aux utilisateurs de se connecter au serveur, d'envoyer des messages et de recevoir des messages des autres utilisateurs. Les concepts clés tels que les threads, Swing et les sockets sont utilisés pour réaliser la communication en réseau et l'interface utilisateur graphique. Ces codes peuvent servir de point de départ pour développer des fonctionnalités plus avancées dans une application de chat.
