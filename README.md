import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.net.*;

class user1 extends Frame implements Runnable, ActionListener {
    TextField textField;
    TextArea textArea;
    Button send;

    ServerSocket serverSocket;
    Socket socket;
    DataInputStream dataInputStream;
    DataOutputStream dataOutputStream;

    Thread chat;

    user1() {

        setTitle("User1");
        setSize(500, 500);
        setLayout(new BorderLayout());


        textArea = new TextArea();
        textArea.setEditable(false);
        textArea.setFont(new Font("Arial", Font.PLAIN, 14));

        textField = new TextField();
        textField.setFont(new Font("Arial", Font.PLAIN, 14));

        send = new Button("Send");
        send.setBackground(new Color(100, 150, 255));
        send.setForeground(Color.WHITE);
        send.setFont(new Font("Arial", Font.BOLD, 14));


        Panel textPanel = new Panel(new BorderLayout());
        ScrollPane scrollPane = new ScrollPane();
        scrollPane.add(textArea);
        textPanel.add(scrollPane, BorderLayout.CENTER);

        Panel inputPanel = new Panel(new BorderLayout());
        inputPanel.add(textField, BorderLayout.CENTER);
        inputPanel.add(send, BorderLayout.EAST);

        add(textPanel, BorderLayout.CENTER);
        add(inputPanel, BorderLayout.SOUTH);

        send.addActionListener(this);
        textField.addActionListener(this);

        try {
            serverSocket = new ServerSocket(1200);
            textArea.append("Waiting for user2 to connect...\n");
            socket = serverSocket.accept();
            textArea.append("User2 connected!\n");

            dataInputStream = new DataInputStream(socket.getInputStream());
            dataOutputStream = new DataOutputStream(socket.getOutputStream());
        } catch (IOException e) {
            textArea.append("Connection Error: " + e.getMessage());
        }

        chat = new Thread(this);
        chat.start();

        setVisible(true);
        setResizable(false);
        addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent e) {
                try {
                    dataInputStream.close();
                    dataOutputStream.close();
                    socket.close();
                    serverSocket.close();
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
                System.exit(0);
            }
        });
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        String msg = textField.getText();
        if (!msg.trim().isEmpty()) {
            textArea.append("You: " + msg + "\n");
            textField.setText("");

            try {
                dataOutputStream.writeUTF(msg);
                dataOutputStream.flush();
            } catch (IOException ex) {
                textArea.append("Failed to send message.\n");
            }
        }
    }

    public static void main(String[] args) {
        new user1();
    }

    @Override
    public void run() {
        while (true) {
            try {
                String msg = dataInputStream.readUTF();
                textArea.append("User2: " + msg + "\n");
            } catch (Exception e) {
                textArea.append("Connection Lost.\n");
                break;
            }
        }
    }
}
import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.net.*;

class user2 extends Frame implements Runnable, ActionListener {
    TextField textField;
    TextArea textArea;
    Button send;

    Socket socket;
    DataInputStream dataInputStream;
    DataOutputStream dataOutputStream;

    Thread chat;

    user2() {
        setTitle("Chat App - User2");
        setSize(500, 500);
        setLayout(new BorderLayout());

        textArea = new TextArea();
        textArea.setEditable(false);
        textArea.setFont(new Font("Arial", Font.PLAIN, 14));

        textField = new TextField();
        textField.setFont(new Font("Arial", Font.PLAIN, 14));

        send = new Button("Send");
        send.setBackground(new Color(100, 150, 255));
        send.setForeground(Color.WHITE);
        send.setFont(new Font("Arial", Font.BOLD, 14));

        Panel textPanel = new Panel(new BorderLayout());
        ScrollPane scrollPane = new ScrollPane();
        scrollPane.add(textArea);
        textPanel.add(scrollPane, BorderLayout.CENTER);

        Panel inputPanel = new Panel(new BorderLayout());
        inputPanel.add(textField, BorderLayout.CENTER);
        inputPanel.add(send, BorderLayout.EAST);

        add(textPanel, BorderLayout.CENTER);
        add(inputPanel, BorderLayout.SOUTH);

        send.addActionListener(this);
        textField.addActionListener(this);

        try {
            textArea.append("Connecting to User1...\n");
            socket = new Socket("localhost", 1200); // Connect to server running at localhost on port 1200
            textArea.append("Connected to User1!\n");

            dataInputStream = new DataInputStream(socket.getInputStream());
            dataOutputStream = new DataOutputStream(socket.getOutputStream());
        } catch (IOException e) {
            textArea.append("Connection Error: " + e.getMessage());
        }

        chat = new Thread(this);
        chat.start();

        setVisible(true);
        setResizable(false);
        addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent e) {
                try {
                    dataInputStream.close();
                    dataOutputStream.close();
                    socket.close();
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
                System.exit(0);
            }
        });
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        String msg = textField.getText();
        if (!msg.trim().isEmpty()) {
            textArea.append("You: " + msg + "\n");
            textField.setText("");

            try {
                dataOutputStream.writeUTF(msg);
                dataOutputStream.flush();
            } catch (IOException ex) {
                textArea.append("Failed to send message.\n");
            }
        }
    }

    public static void main(String[] args) {
        new user2();
    }

    @Override
    public void run() {
        while (true) {
            try {
                String msg = dataInputStream.readUTF();
                textArea.append("User1: " + msg + "\n");
            } catch (Exception e) {
                textArea.append("Connection Lost.\n");
                break;
            }
        }
    }
}
