# Library-management-system
import javax.swing.*;
import javax.swing.border.EmptyBorder;
import javax.swing.table.DefaultTableModel;
import java.awt.*;
import java.awt.event.*;
import java.sql.*;

public class Main extends JFrame {
    private JTextField nameField, authorField, searchField;
    private JTable bookTable;
    private DefaultTableModel tableModel;

    public Main() {
        setTitle("Book Library");
        setSize(900, 600);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Load the background image
        ImageIcon backgroundIcon = new ImageIcon("omg1.jpg");
        Image backgroundImage = backgroundIcon.getImage();

        // Background panel
        JPanel backgroundPanel = new JPanel() {
            @Override
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                g.drawImage(backgroundImage, 0, 0, getWidth(), getHeight(), this);
            }
        };
        backgroundPanel.setLayout(new BorderLayout());
        backgroundPanel.setBorder(new EmptyBorder(10, 10, 10, 10));

        // Colors and fonts
        Color darkViolet = new Color(48, 25, 52, 200); // semi-transparent
        Color white = Color.WHITE;
        Font mainFont = new Font("Segoe UI", Font.PLAIN, 16);
        Font headerFont = new Font("Segoe UI", Font.BOLD, 22);

        // NAVBAR
        JPanel navBar = new JPanel(new BorderLayout());
        navBar.setBackground(new Color(32, 18, 40, 200));
        navBar.setPreferredSize(new Dimension(900, 50));

        JLabel title = new JLabel("  Book Library", SwingConstants.LEFT);
        title.setForeground(white);
        title.setFont(new Font("Segoe UI", Font.BOLD, 20));
        navBar.add(title, BorderLayout.WEST);

        JPanel searchPanel = new JPanel();
        searchPanel.setOpaque(false);
        searchField = new JTextField(15);
        JButton searchButton = new JButton("Search");
        styleButton(searchButton, new Color(102, 0, 153), white, mainFont);
        searchPanel.add(searchField);
        searchPanel.add(searchButton);
        navBar.add(searchPanel, BorderLayout.EAST);

        backgroundPanel.add(navBar, BorderLayout.NORTH);

        // FORM SECTION
        JPanel formPanel = new JPanel();
        formPanel.setOpaque(false);
        formPanel.setLayout(new BoxLayout(formPanel, BoxLayout.Y_AXIS));
        formPanel.setBorder(new EmptyBorder(20, 0, 10, 0));

        JLabel manageLabel = new JLabel("Manage Library");
        manageLabel.setFont(headerFont);
        manageLabel.setForeground(white);
        formPanel.add(manageLabel);

        nameField = createTextField(mainFont);
        JLabel nameLabel = createLabel("Name", mainFont, white);
        formPanel.add(nameLabel);
        formPanel.add(nameField);

        authorField = createTextField(mainFont);
        JLabel authorLabel = createLabel("Author", mainFont, white);
        formPanel.add(authorLabel);
        formPanel.add(authorField);

        JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.LEFT));
        buttonPanel.setOpaque(false);
        JButton addButton = new JButton("Add Book");
        styleButton(addButton, new Color(102, 0, 204), white, mainFont);
        JButton deleteButton = new JButton("Delete Selected");
        styleButton(deleteButton, new Color(178, 34, 34), white, mainFont);
        buttonPanel.add(addButton);
        buttonPanel.add(deleteButton);
        formPanel.add(Box.createVerticalStrut(10));
        formPanel.add(buttonPanel);

        backgroundPanel.add(formPanel, BorderLayout.CENTER);

        // TABLE SECTION
        JLabel booksLabel = new JLabel("Your books");
        booksLabel.setFont(headerFont);
        booksLabel.setForeground(white);
        booksLabel.setBorder(new EmptyBorder(10, 0, 5, 0));

        String[] columns = {"Name", "Author"};
        tableModel = new DefaultTableModel(columns, 0);
        bookTable = new JTable(tableModel);
        bookTable.setFont(mainFont);
        bookTable.setRowHeight(25);
        bookTable.getTableHeader().setFont(mainFont);

        JScrollPane tableScroll = new JScrollPane(bookTable);

        JPanel tablePanel = new JPanel(new BorderLayout());
        tablePanel.setOpaque(false);
        tablePanel.add(booksLabel, BorderLayout.NORTH);
        tablePanel.add(tableScroll, BorderLayout.CENTER);

        backgroundPanel.add(tablePanel, BorderLayout.SOUTH);

        // Add to frame
        setContentPane(backgroundPanel);

        // Load books
        loadBooks();

        // Add functionality
        addButton.addActionListener(e -> {
            String name = nameField.getText().trim();
            String author = authorField.getText().trim();
            if (!name.isEmpty() && !author.isEmpty()) {
                addBook(name, author);
                tableModel.addRow(new Object[]{name, author});
                nameField.setText("");
                authorField.setText("");
            }
        });

        deleteButton.addActionListener(e -> {
            int selectedRow = bookTable.getSelectedRow();
            if (selectedRow >= 0) {
                String name = tableModel.getValueAt(selectedRow, 0).toString();
                String author = tableModel.getValueAt(selectedRow, 1).toString();
                deleteBook(name, author);
                tableModel.removeRow(selectedRow);
            } else {
                JOptionPane.showMessageDialog(this, "Please select a book to delete.");
            }
        });

        searchButton.addActionListener(e -> {
            String keyword = searchField.getText().toLowerCase();
            bookTable.clearSelection();
            for (int i = 0; i < tableModel.getRowCount(); i++) {
                boolean match = false;
                for (int j = 0; j < tableModel.getColumnCount(); j++) {
                    Object value = tableModel.getValueAt(i, j);
                    if (value != null && value.toString().toLowerCase().contains(keyword)) {
                        match = true;
                        break;
                    }
                }
                if (match) {
                    bookTable.addRowSelectionInterval(i, i);
                }
            }
        });
    }

    private void styleButton(JButton button, Color bg, Color fg, Font font) {
        button.setBackground(bg);
        button.setForeground(fg);
        button.setFocusPainted(false);
        button.setFont(font);
    }

    private JTextField createTextField(Font font) {
        JTextField field = new JTextField();
        field.setMaximumSize(new Dimension(Integer.MAX_VALUE, 30));
        field.setFont(font);
        return field;
    }

    private JLabel createLabel(String text, Font font, Color color) {
        JLabel label = new JLabel(text);
        label.setFont(font);
        label.setForeground(color);
        return label;
    }

    private Connection getConnection() throws SQLException {
        return DriverManager.getConnection("jdbc:mysql://localhost:3306/library", "root", "root");
    }

    private void loadBooks() {
        try (Connection conn = getConnection()) {
            String query = "SELECT * FROM books";
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery(query);
            while (rs.next()) {
                String name = rs.getString("name");
                String author = rs.getString("author");
                tableModel.addRow(new Object[]{name, author});
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private void addBook(String name, String author) {
        try (Connection conn = getConnection()) {
            String query = "INSERT INTO books (name, author) VALUES (?, ?)";
            PreparedStatement stmt = conn.prepareStatement(query);
            stmt.setString(1, name);
            stmt.setString(2, author);
            stmt.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private void deleteBook(String name, String author) {
        try (Connection conn = getConnection()) {
            String query = "DELETE FROM books WHERE name = ? AND author = ?";
            PreparedStatement stmt = conn.prepareStatement(query);
            stmt.setString(1, name);
            stmt.setString(2, author);
            stmt.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new Main().setVisible(true));
    }
}
