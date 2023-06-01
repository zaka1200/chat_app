# chat_app

# Serveur

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
    int port = 12345;
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

        // Effectuer l'authentification
        writer.println("Entrez votre nom d'utilisateur :");
        username = reader.readLine();

        server.broadcastMessage(username + " a rejoint le chat.", this);

        String clientMessage;
        while ((clientMessage = reader.readLine()) != null) {
            server.broadcastMessage(clientMessage, this);
        }

        socket.close();
        server.removeClient(this);
        server.broadcastMessage(username + " a quitté le chat.", this);
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
