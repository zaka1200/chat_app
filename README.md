# chat_app

# Introduction :

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

# Client :

#

Le code `ChatClient` représente le client de l'application de chat. Il utilise un `Socket` pour se connecter au serveur. Une fois connecté, le client peut envoyer des messages au serveur et recevoir des messages des autres utilisateurs connectés. Le client utilise également l'interface graphique Swing pour créer une fenêtre de chat conviviale. Il utilise des composants tels que `JTextArea`, `JTextField` et `JButton` pour afficher les messages, saisir les messages et envoyer les messages respectivement.

Lorsqu'un utilisateur envoie un message, le client l'envoie au serveur via le `Socket` et l'affiche dans la zone de chat en utilisant `JTextArea`. Le client utilise également un mécanisme de couleur pour afficher les messages des utilisateurs dans des couleurs différentes, en utilisant `Color` de Swing.

#

```java
import java.io.*;
import java.net.*;
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.HashMap;
import java.util.Map;

public class ChatClient {
    private Socket socket;
    private BufferedReader reader;
    private PrintWriter writer;
    private JTextArea chatArea;
    private JTextField messageField;
    private String username;
    private Map<String, Color> userColors;

    public ChatClient(String serverAddress, int serverPort) {
        try {
            socket = new Socket(serverAddress, serverPort);
            reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            writer = new PrintWriter(socket.getOutputStream(), true);
            userColors = new HashMap<>();
            performAuthentication();
            createAndShowGUI();
            startReadingMessages();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

La classe `ChatClient` représente le côté client de l'application de chat. Elle a plusieurs variables d'instance, notamment `socket` de type `Socket`, `reader` et `writer` de type `BufferedReader` et `PrintWriter` respectivement pour la communication avec le serveur, `chatArea` et `messageField` de type `JTextArea` et `JTextField` respectivement pour l'interface utilisateur graphique (GUI), `username` pour stocker le nom d'utilisateur du client et `userColors` de type `Map<String, Color>` pour stocker les couleurs des utilisateurs.

Le constructeur `ChatClient` crée une connexion socket vers le serveur spécifié, initialise les flux de lecture et d'écriture, et effectue l'authentification du client en appelant `performAuthentication()`. Ensuite, il crée et affiche l'interface utilisateur graphique en appelant `createAndShowGUI()`, et démarre la lecture des messages du serveur en appelant `startReadingMessages()`.

```java
public void createAndShowGUI() {
    // Définir l'apparence selon le style visuel du système
    try {
        UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
    } catch (Exception ex) {
        ex.printStackTrace();
    }

    // Créer les composants de l'interface utilisateur
    chatArea = new JTextArea(20, 40);
    chatArea.setEditable(false);

    messageField = new JTextField(40);

    JButton sendButton = new JButton("Envoyer");
    sendButton.addActionListener(new ActionListener() {
        @Override
        public void actionPerformed(ActionEvent e) {
            String message = messageField.getText();
            sendMessage(message);
            appendMessage("Vous", message, true);
            messageField.setText("");
        }
    });

    // Configurer la disposition des composants
    JPanel chatPanel = new JPanel(new BorderLayout());
    chatPanel.add(new JScrollPane(chatArea), BorderLayout.CENTER);

    JPanel inputPanel = new JPanel(new BorderLayout());
    inputPanel.add(messageField, BorderLayout.CENTER);
    inputPanel.add(sendButton, BorderLayout.EAST);

    JPanel mainPanel = new JPanel(new BorderLayout());
    mainPanel.add(chatPanel, BorderLayout.CENTER);
    mainPanel.add(inputPanel, BorderLayout.SOUTH);

    // Configurer la fenêtre
    JFrame frame = new JFrame("Client de Chat");
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    frame.getContentPane().add(mainPanel);
    frame.setSize(600, 400); // Définir la taille de la fenêtre à 600x400 pixels
    frame.setLocationRelativeTo(null);
    frame.setVisible(true);
}
```

La méthode `createAndShowGUI` est responsable de la création et de l

'affichage de l'interface utilisateur graphique (GUI) du client. Elle utilise les composants Swing pour créer une fenêtre de chat basique.

Elle crée un `JTextArea` pour afficher les messages du chat, un `JTextField` pour permettre au client de saisir des messages, et un bouton "Envoyer" pour envoyer les messages saisis. Lorsque le bouton "Envoyer" est cliqué, il appelle la méthode `sendMessage` pour envoyer le message saisi au serveur et la méthode `appendMessage` pour afficher le message dans la zone de chat.

La méthode configure les différents panneaux (panels) pour organiser les composants de l'interface utilisateur graphique, crée une fenêtre `JFrame` et l'affiche à l'écran.

```java
private void sendMessage(String message) {
    writer.println(message);
}

private void appendMessage(String sender, String message, boolean isLocalUser) {
    String timestamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss"));
    String formattedMessage = "[" + timestamp + "] " + sender + ": " + message;

    Color messageColor;
    if (isLocalUser) {
        messageColor = Color.BLUE; // Couleur pour les messages envoyés par le client local
    } else {
        messageColor = getUserColor(sender);
    }

    StyleContext styleContext = StyleContext.getDefaultStyleContext();
    AttributeSet attributes = styleContext.addAttribute(SimpleAttributeSet.EMPTY, StyleConstants.Foreground, messageColor);

    int length = chatArea.getDocument().getLength();
    chatArea.setCaretPosition(length);
    chatArea.setCharacterAttributes(attributes, false);
    chatArea.replaceSelection(formattedMessage + "\n");
}
```

La méthode `sendMessage` envoie le message spécifié au serveur en utilisant le `PrintWriter`.

La méthode `appendMessage` ajoute un nouveau message à la zone de chat avec des détails tels que l'expéditeur, le message et l'heure. Elle utilise un `LocalDateTime` pour obtenir l'heure actuelle, formate le message en conséquence, définit la couleur du message (bleu pour les messages du client local et une couleur générée aléatoirement pour les autres utilisateurs), et affiche le message formaté dans la zone de chat en utilisant `chatArea.replaceSelection`.

```java
private Color getUserColor(String username) {
    if (userColors.containsKey(username)) {
        return userColors.get(username);
    } else {
        Color color = generateRandomColor();
        userColors.put(username, color);
        return color;
    }
}

private Color generateRandomColor() {
    int r = (int) (Math.random() * 256);
    int g = (int) (Math.random() * 256);
    int b = (int) (Math.random() * 256);
    return new Color(r, g, b);
}
```

La méthode `getUserColor` retourne la couleur associée à un nom d'utilisateur spécifique. Si la couleur n'est pas encore définie pour cet utilisateur, elle génère une couleur aléatoire en utilisant `generateRandomColor`, l'associe à l'utilisateur dans la `Map` `userColors`, puis la retourne.

La méthode `generateRandomColor` génère une couleur aléatoire en utilisant des valeurs de composantes RVB (rouge, vert, bleu) aléatoires.

```java
private void performAuthentication() throws IOException {
    username = JOptionPane.showInputDialog(null, "Entrez votre nom d'utilisateur :");
    writer.println(username);
}
```

La méthode `perform

Authentication` effectue l'authentification du client en demandant à l'utilisateur de saisir son nom d'utilisateur à l'aide d'une boîte de dialogue `JOptionPane`, puis envoie ce nom d'utilisateur au serveur en utilisant `writer.println`.

```java
private void startReadingMessages() {
    Thread readThread = new Thread(() -> {
        try {
            String message;
            while ((message = reader.readLine()) != null) {
                String[] parts = message.split(": ", 2);
                if (parts.length == 2) {
                    String sender = parts[0];
                    String actualMessage = parts[1];
                    boolean isLocalUser = sender.equals(username);
                    appendMessage(sender, actualMessage, isLocalUser);
                    playNotificationSound();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    });
    readThread.start();
}
```

La méthode `startReadingMessages` démarre un nouveau thread pour lire les messages du serveur en continu. Elle lit les messages ligne par ligne à partir du `BufferedReader` `reader`. Chaque message est analysé en utilisant `split` pour extraire l'expéditeur et le message réel. Ensuite, la méthode `appendMessage` est appelée pour afficher le message dans la zone de chat avec une indication si le message provient du client local ou d'un autre utilisateur. Enfin, la méthode `playNotificationSound` est appelée pour jouer un son de notification.

```java
private void playNotificationSound() {
    // Implémentez la logique pour jouer un son de notification
    // lorsqu'un nouveau message est reçu.
    // Vous pouvez utiliser des bibliothèques telles que javax.sound ou JavaFX Media pour cela.
}

public static void main(String[] args) {
    String serverAddress = "localhost"; // Remplacez par l'adresse de votre serveur
    int serverPort = 12345; // Remplacez par le port de votre serveur

    SwingUtilities.invokeLater(() -> new ChatClient(serverAddress, serverPort));
}
```

La méthode `playNotificationSound` est un espace réservé pour implémenter la logique de lecture d'un son de notification lorsqu'un nouveau message est reçu. Vous pouvez utiliser des bibliothèques telles que `javax.sound` ou JavaFX Media pour cela.

La méthode `main` est la méthode d'entrée du programme. Elle crée une instance de `ChatClient` en spécifiant l'adresse et le port du serveur, et le fait dans le thread de l'interface utilisateur en utilisant `SwingUtilities.invokeLater`.

# Threads :

Les threads jouent un rôle crucial dans ces deux codes. Dans le `ChatServer`, chaque connexion client est gérée par un thread `ClientHandler` distinct. Cela permet au serveur de gérer plusieurs connexions simultanément et de répondre aux messages de chaque client sans bloquer les autres clients. De même, dans le `ChatClient`, un thread distinct est utilisé pour lire les messages du serveur en continu tout en maintenant l'interface utilisateur réactive.

# Swing :

Swing est une bibliothèque graphique en Java qui permet de créer des interfaces graphiques. Dans le `ChatClient`, Swing est utilisé pour créer une fenêtre de chat conviviale avec des composants tels que `JTextArea`, `JTextField` et `JButton`. Swing fournit également des fonctionnalités pour la gestion des événements, telles que l'ajout d'un écouteur d'action au bouton "Envoyer". L'utilisation de Swing facilite la création d'une interface utilisateur interactive et réactive pour le client.

# Sockets :

Les sockets sont utilisés pour établir une communication réseau entre le serveur et les clients. Le serveur utilise une `ServerSocket` pour écouter les connexions entrantes des clients, tandis que chaque client utilise un `Socket` pour se connecter au serveur. Les sockets permettent aux données d'être envoyées et reçues via le réseau, facilitant ainsi la communication bidirectionnelle entre le serveur et les clients.

# Test :


# Consclusion :


En conclusion, les codes `ChatServer` et `ChatClient` présentent une implémentation basique d'une application de chat en Java. Le `ChatServer` gère les connexions des clients, la diffusion des messages et les déconnexions, tandis que le `ChatClient` permet aux utilisateurs de se connecter au serveur, d'envoyer des messages et de recevoir des messages des autres utilisateurs. Les concepts clés tels que les threads, Swing et les sockets sont utilisés pour réaliser la communication en réseau et l'interface utilisateur graphique. Ces codes peuvent servir de point de départ pour développer des fonctionnalités plus avancées dans une application de chat.
