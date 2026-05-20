CREATE DATABASE IF NOT EXISTS sistema_login;
USE sistema_login;

-- Tabla Padre: Usuarios
CREATE TABLE usuarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    usuario VARCHAR(50) NOT NULL UNIQUE,
    clave VARCHAR(255) NOT NULL
);

-- Tabla Hija: Historial de Ingresos (Para cumplir el Criterio 2)
CREATE TABLE ingresos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    usuario_id INT NOT NULL,
    fecha_ingreso DATETIME NOT NULL,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE CASCADE
);

-- Insertar un usuario de prueba (Clave: admin123)
INSERT INTO usuarios (usuario, clave) VALUES ('admin', 'admin123'); package alumnos;

package config;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class Conexion {
    private static final String URL = "jdbc:mysql://localhost:3306/sistema_login";
    private static final String USER = "root";
    private static final String PASSWORD = ""; // Cambia si usas contraseña en XAMPP

    public static Connection getConexion() {
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            return DriverManager.getConnection(URL, USER, PASSWORD);
        } catch (ClassNotFoundException | SQLException e) {
            System.out.println("Error de conexión: " + e.getMessage());
            return null;
        }
    }
}

package model;

public class Usuario {
    private int id;
    private String usuario;
    private String clave;

    public Usuario() {}

    public Usuario(int id, String usuario, String clave) {
        this.id = id;
        this.usuario = usuario;
        this.clave = clave;
    }

    // Getters y Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getUsuario() { return usuario; }
    public void setUsuario(String usuario) { this.usuario = usuario; }
    public String getClave() { return clave; }
    public void setClave(String clave) { this.clave = clave; }
} 

package dao;

import config.Conexion;
import model.Usuario;
import java.sql.*;
import java.time.LocalDateTime;

public class UsuarioDAO {

    // Retorna el objeto Usuario si es válido, de lo contrario devuelve null
    public Usuario validarLogin(String txtUsuario, String txtClave) {
        String sql = "SELECT * FROM usuarios WHERE usuario = ? AND clave = ?";
        try (Connection con = Conexion.getConexion();
             PreparedStatement ps = con.prepareStatement(sql)) {
            
            ps.setString(1, txtUsuario);
            ps.setString(2, txtClave);
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    return new Usuario(rs.getInt("id"), rs.getString("usuario"), rs.getString("clave"));
                }
            }
        } catch (SQLException e) {
            System.out.println("Error al validar: " + e.getMessage());
        }
        return null;
    }

    // Registra el ingreso en la tabla hija
    public boolean registrarIngreso(int usuarioId) {
        String sql = "INSERT INTO ingresos (usuario_id, fecha_ingreso) VALUES (?, ?)";
        try (Connection con = Conexion.getConexion();
             PreparedStatement ps = con.prepareStatement(sql)) {
            
            ps.setInt(1, usuarioId);
            // Uso de LocalDateTime.now() solicitado en la pizarra
            ps.setTimestamp(2, Timestamp.valueOf(LocalDateTime.now())); 
            
            return ps.executeUpdate() > 0;
        } catch (SQLException e) {
            System.out.println("Error al registrar ingreso: " + e.getMessage());
            return false;
        }
    }
}

private void btnLoginActionPerformed(java.awt.event.ActionEvent evt) {                                         
    String txtUser = txtUsuario.getText().trim();
    String txtPass = new String(txtPassword.getPassword()).trim();

    // Validar que los campos no estén vacíos
    if (txtUser.isEmpty() || txtPass.isEmpty()) {
        JOptionPane.showMessageDialog(this, "Por favor, llene todos los campos.");
        return;
    }

    UsuarioDAO dao = new UsuarioDAO();
    model.Usuario user = dao.validarLogin(txtUser, txtPass);

    if (user != null) {
        // 1. Mensaje de éxito exacto pedido en el examen
        JOptionPane.showMessageDialog(this, "Bienvenido al sistema Usuario " + user.getUsuario());
        
        // 2. Registrar el ingreso en la base de datos (Criterio 2)
        dao.registrarIngreso(user.getId());
        
        // 3. Permitir el ingreso de otro usuario (Limpiar campos para el siguiente)
        txtUsuario.setText("");
        txtPassword.setText("");
        txtUsuario.requestFocus();
        
    } else {
        // Mensaje de error exacto pedido en el examen
        JOptionPane.showMessageDialog(this, "Usuario o clave incorrecta");
    }
}
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.ArrayList;
import java.util.List;

public class Interfa {
    public static void main(String[] args) {
        try {
            String rArt = "C:\\Documentos\\NetBeansProjects\\Libros\\src\\libros\\Libro.java";
            String cod = Files.readString(Path.of(rArt));

            // p1 contiene la instrucción compactada y la estructura SQL pegada al código Java
            String p1 = "p1"+ cod;

            Path rTmp = Path.of("C:\\Users\\Ariel\\Downloads\\DD\\llama-b9093-bin-win-cpu-x64\\prompt_temp.txt");
            Files.writeString(rTmp, p1);

            String dir = "C:\\Users\\Ariel\\Downloads\\DD\\llama-b9093-bin-win-cpu-x64\\";
            
            List<String> cmd = new ArrayList<>();
            cmd.add(dir + "l.exe");
            cmd.add("-m");
            cmd.add(dir + "m.gguf");
            cmd.add("-c");
            cmd.add("8192"); 
            cmd.add("-f");    
            cmd.add(rTmp.toString()); 

            System.out.println("...");
            
            ProcessBuilder pb = new ProcessBuilder(cmd);
            pb.redirectErrorStream(true); 
            Process prc = pb.start();

            BufferedReader lec = new BufferedReader(new InputStreamReader(prc.getInputStream(), "UTF-8"));
            String lin;

            System.out.println("\n------");
            while ((lin = lec.readLine()) != null) {
                if (lin.contains("load_backend") || lin.contains("llama_model_loader") || lin.contains("llm_load_print_meta")) {
                    continue;
                }
                System.out.println(lin);
            }

            prc.waitFor();
            lec.close();
            Files.deleteIfExists(rTmp);
            
            System.out.println("-----------");

        } catch (Exception e) {
            System.err.println("Err: " + e.getMessage());
        }
    }
}


package alumnos;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class pp {
    private static Process srv = null;

    public static void main(String[] args) {
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            if (srv != null && srv.isAlive()) srv.destroy();
        }));

        try {
            String dir = "C:\\Users\\Ariel\\Downloads\\DD\\llama-b9093-bin-win-cpu-x64\\";
            
            List<String> cmd = new ArrayList<>();
            cmd.add(dir + "llama-server.exe");
            cmd.add("-m");
            cmd.add(dir + "m.gguf");
            cmd.add("-c");
            cmd.add("8192");
            cmd.add("--port");
            cmd.add("5000");

            ProcessBuilder pb = new ProcessBuilder(cmd);
            pb.redirectError(ProcessBuilder.Redirect.DISCARD);
            pb.redirectOutput(ProcessBuilder.Redirect.DISCARD);
            srv = pb.start();

            Thread.sleep(8000); // Subido a 8 segundos para asegurar el arranque limpio

            Scanner sc = new Scanner(System.in, "UTF-8");
            HttpClient cl = HttpClient.newHttpClient();

            while (true) {
                System.out.print("\n> ");
                String p = sc.nextLine();

                if (p.equalsIgnoreCase("salir")) break;
                if (p.trim().isEmpty()) continue;

                String pEsc = p.replace("\r", "").replace("\"", "\\\"").replace("\n", "\\n");
                String body = "{\"prompt\": \"" + pEsc + "\", \"n_predict\": 1000}";

                HttpRequest req = HttpRequest.newBuilder()
                        .uri(URI.create("http://localhost:5000/completion"))
                        .header("Content-Type", "application/json")
                        .POST(HttpRequest.BodyPublishers.ofString(body))
                        .build();

                System.out.println("...");
                try {
                    HttpResponse<String> res = cl.send(req, HttpResponse.BodyHandlers.ofString());
                    System.out.println(res.body());
                    System.out.println("---");
                } catch (Exception e) {
                    System.err.println("Error de conexion. Revisa si el puerto 5000 quedo bloqueado.");
                }
            }
        } catch (Exception e) {
            System.err.println("Err: " + e.getMessage());
        } finally {
            if (srv != null && srv.isAlive()) srv.destroy();
        }
    }
}



package alumnos;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.file.Files;
import java.nio.file.Path;

public class probar {
    public static void main(String[] args) {
        try {
            // Ajustado a la ruta de tu proyecto "Libros"
            String carpetaProyecto = "C:\\Documentos\\NetBeansProjects\\Libros\\src\\libros\\";
            
            String[] archivosAnalizar = {
                "Interf.java",
                "DAO.java",
                "Libro.java",
                "Autor.java"
            };

            StringBuilder codigoConsolidado = new StringBuilder();
            for (String nombreArchivo : archivosAnalizar) {
                Path rutaCompleta = Path.of(carpetaProyecto + nombreArchivo);
                if (Files.exists(rutaCompleta)) {
                    // Usamos \n normal aquí para que el formateador global trabaje limpio
                    codigoConsolidado.append("=== ARCHIVO: ").append(nombreArchivo).append(" ===\n");
                    codigoConsolidado.append(Files.readString(rutaCompleta));
                    codigoConsolidado.append("\n=== FIN DE ARCHIVO ===\n\n");
                }
            }

            // Formateo global para el cuerpo del JSON
            String codigoEscapado = codigoConsolidado.toString()
                    .replace("\r", "")
                    .replace("\"", "\\\"")
                    .replace("\n", "\\n");

            // CORRECCIÓN: Separamos la instrucción del código con saltos de línea legibles para la IA
            String prompt = "Analiza estos 4 archivos y dime como en Interf.java añado los valores de la lista de DAO.java y en una lista de la GUI:\\n\\n" + codigoEscapado;
            
            String jsonBody = "{\"prompt\": \"" + prompt + "\", \"n_predict\": 1000}";

            System.out.println("Enviando paquete de auditoria al servicio local...");

            HttpClient client = HttpClient.newHttpClient();
            HttpRequest request = HttpRequest.newBuilder()
                    .uri(URI.create("http://localhost:5000/completion"))
                    .header("Content-Type", "application/json")
                    .POST(HttpRequest.BodyPublishers.ofString(jsonBody))
                    .build();

            HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
            
            System.out.println("\n--");
            System.out.println(response.body());
            System.out.println("------------------------------------");

        } catch (Exception e) {
            System.err.println("Error: " + e.getMessage());
            e.printStackTrace();
        }
    }
}


package alumnos;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class prueba {
    public static void main(String[] args) {
        try {
            String m = "Ok ayudame a crear en la bd una tabla libros y otra autores, donde un libro puede tener un solo autor pero un autor puede tener muchos libros";

            String promptEscapado = m.replace("\r", "")
                                             .replace("\"", "\\\"")
                                             .replace("\n", "\\n");

            String jsonBody = "{\"prompt\": \"" + promptEscapado + "\", \"n_predict\": 1000}";

            HttpClient client = HttpClient.newHttpClient();
            HttpRequest request = HttpRequest.newBuilder()
                    .uri(URI.create("http://localhost:5000/completion"))
                    .header("Content-Type", "application/json")
                    .POST(HttpRequest.BodyPublishers.ofString(jsonBody))
                    .build();

            System.out.println("...");
            HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

            System.out.println(response.body());
            System.out.println("---");

        } catch (Exception e) {
            System.err.println("Error: " + e.getMessage());
        }
    }
}


package libros;

public class Autor {
    private int id;
    private String nombre;
    
    public Autor(int id, String nombre){
        this.id = id;
        this.nombre = nombre;
    }

    // AGREGA ESTOS DOS GETTERS CRÍTICOS:
    public int getId() { return id; }
    public String getNombre() { return nombre; }
}


package libros;

public class Libro {
    private int id;
    private String nombre;
    private int autorId;

    public Libro(int id, String nombre, int autorId) {
        this.id = id;
        this.nombre = nombre;
        this.autorId = autorId;
    }

    // Asegúrate de tener estos tres getters:
    public int getId() { return id; }
    public String getNombre() { return nombre; }
    public int getAutorId() { return autorId; }
}


package libros;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class DAO {

    public static List<Libro> traerLibros() {
        List<Libro> libros = new ArrayList<>();
        
        String url = "jdbc:mysql://localhost:3306/prueba"; 
        String user = "root"; 
        String password = ""; 

        try (Connection conn = DriverManager.getConnection(url, user, password);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT id, nombre, autor_id FROM libros")) {
            
            while (rs.next()) {
                int id = rs.getInt("id");
                String nombre = rs.getString("nombre");
                int autorId = rs.getInt("autor_id");
                
                Libro libro = new Libro(id, nombre, autorId);
                libros.add(libro);
            }
            
        } catch (SQLException e) {
            System.out.println("Error en la BD: " + e.getMessage());
        }
        return libros;
    }
    
    public static List<Autor> traerAutores() {
        List<Autor> autores = new ArrayList<>();
        String url = "jdbc:mysql://localhost:3306/prueba"; 
        String user = "root"; 
        String password = ""; 

        // Consulta simple a la tabla autores
        try (Connection conn = DriverManager.getConnection(url, user, password);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT id, nombre FROM autores")) {
            
            while (rs.next()) {
                int id = rs.getInt("id");
                String nombre = rs.getString("nombre");
                
                // Creamos el objeto Autor y lo metemos a la lista
                Autor autor = new Autor(id, nombre);
                autores.add(autor);
            }
            
        } catch (SQLException e) {
            System.out.println("Error en la BD al traer autores: " + e.getMessage());
        }
        return autores;
    }
    
    public static boolean guardarLibro(String nombreLibro, int autorId) {
        String url = "jdbc:mysql://localhost:3306/prueba"; 
        String user = "root"; 
        String password = ""; 
        
        // Usamos ? para evitar inyección SQL (buenas prácticas de seguridad)
        String sql = "INSERT INTO libros (nombre, autor_id) VALUES (?, ?)";

        try (Connection conn = DriverManager.getConnection(url, user, password);
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            // Reemplazamos los signos de interrogación por los datos reales
            pstmt.setString(1, nombreLibro);
            pstmt.setInt(2, autorId);
            
            // Ejecuta la inserción en la base de datos
            int filasAfectadas = pstmt.executeUpdate();
            return filasAfectadas > 0; // Devuelve true si se guardó con éxito
            
        } catch (SQLException e) {
            System.out.println("Error al guardar el libro: " + e.getMessage());
            return false;
        }
    }
    
    public static boolean guardarAutor(String nombreAutor) {
        String url = "jdbc:mysql://localhost:3306/prueba"; 
        String user = "root"; 
        String password = ""; 
        
        String sql = "INSERT INTO autores (nombre) VALUES (?)";

        try (Connection conn = DriverManager.getConnection(url, user, password);
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, nombreAutor);
            
            int filasAfectadas = pstmt.executeUpdate();
            return filasAfectadas > 0; // true si se guardó correctamente
            
        } catch (SQLException e) {
            System.out.println("Error al guardar el autor: " + e.getMessage());
            return false;
        }
    }
}


/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 * Click nbfs://nbhost/SystemFileSystem/Templates/GUIForms/JFrame.java to edit this template
 */
package libros;

import javax.swing.DefaultListModel;
import java.util.List;
import javax.swing.JOptionPane;
import javax.swing.table.DefaultTableModel;

/**
 *
 * @author Ariel
 */
public class Interf extends javax.swing.JFrame {
    
    private static final java.util.logging.Logger logger = java.util.logging.Logger.getLogger(Interf.class.getName());

    /**
     * Creates new form Libro
     */
    public Interf() {
        initComponents();
    }

    public void conectarse() {
        List<Libro> bd = DAO.traerLibros();
        DefaultListModel<String> modeloLista = new DefaultListModel<>();
        
        for (Libro libro : bd) {
            modeloLista.addElement(libro.getNombre());
        }
        jList1.setModel(modeloLista);
    }
    
    public void llenarTabla() {
        List<Libro> db = DAO.traerLibros();
        String[] columnas = {"ID", "Título del Libro", "ID Autor"};
        
        DefaultTableModel modeloTabla = new DefaultTableModel(columnas, 0);
        
        for (Libro libro : db) {
            Object[] fila = {
                libro.getId(),
                libro.getNombre(),
                libro.getAutorId()
            };
            modeloTabla.addRow(fila);
        }
        
        jTable1.setModel(modeloTabla);
    }
    
    public void llenarTablaAutores() {
        // 1. Recuperar los autores usando el nuevo método del DAO
        java.util.List<Autor> listaAutores = DAO.traerAutores();
        
        // 2. Definir las columnas para esta tabla
        String[] columnas = {"ID Autor", "Nombre del Autor"};
        
        // 3. Crear el modelo de la tabla
        DefaultTableModel modeloTabla = new DefaultTableModel(columnas, 0);
        
        // 4. Recorrer la lista de autores y agregarlos como filas
        for (Autor autor : listaAutores) {
            Object[] fila = {
                autor.getId(),
                autor.getNombre()
            };
            modeloTabla.addRow(fila);
        }
        
        // 5. Vincular el modelo a la tabla de autores (jTable2)
        jTable2.setModel(modeloTabla);
    }
    
    private void añadirLibro() {                                             
        String nombreLibro = jTextField1.getText().trim();
        String idAutorTexto = jTextField2.getText().trim();

        if (nombreLibro.isEmpty() || idAutorTexto.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Por favor, llena todos los campos.");
            return;
        }

        try {
           
            int autorId = Integer.parseInt(idAutorTexto);
            boolean exito = DAO.guardarLibro(nombreLibro, autorId);

            if (exito) {
                javax.swing.JOptionPane.showMessageDialog(this, "¡Libro guardado exitosamente!");
   
                jTextField1.setText("");
                jTextField2.setText("");

                llenarTabla(); 
            } else {
                JOptionPane.showMessageDialog(this, "Error al guardar. Verifica que el ID del autor exista.");
            }

        } catch (NumberFormatException e) {
            JOptionPane.showMessageDialog(this, "El ID del Autor debe ser un número válido.");
        }
    }
    
    public void añadirAutor(){
        
        String nombreAutor = jTextField3.getText().trim();

        if (nombreAutor.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Por favor, escribe el nombre del autor.");
            return;
        }

        boolean exito = DAO.guardarAutor(nombreAutor);

        if (exito) {
            JOptionPane.showMessageDialog(this, "¡Autor guardado exitosamente!");
           
            jTextField3.setText("");

            llenarTablaAutores(); 
        } else {
            JOptionPane.showMessageDialog(this, "Error al guardar el autor en la base de datos.");
        }
    }
    /**
     * This method is called from within the constructor to initialize the form.
     * WARNING: Do NOT modify this code. The content of this method is always
     * regenerated by the Form Editor.
     */
    @SuppressWarnings("unchecked")
    // <editor-fold defaultstate="collapsed" desc="Generated Code">                          
    private void initComponents() {

        jLabel1 = new javax.swing.JLabel();
        jLabel2 = new javax.swing.JLabel();
        jTextField1 = new javax.swing.JTextField();
        jTextField2 = new javax.swing.JTextField();
        jButton1 = new javax.swing.JButton();
        jScrollPane1 = new javax.swing.JScrollPane();
        jList1 = new javax.swing.JList<>();
        jScrollPane2 = new javax.swing.JScrollPane();
        jTable1 = new javax.swing.JTable();
        jScrollPane3 = new javax.swing.JScrollPane();
        jTable2 = new javax.swing.JTable();
        jLabel3 = new javax.swing.JLabel();
        jLabel4 = new javax.swing.JLabel();
        jLabel5 = new javax.swing.JLabel();
        jTextField3 = new javax.swing.JTextField();
        jButton2 = new javax.swing.JButton();
        jButton3 = new javax.swing.JButton();

        setDefaultCloseOperation(javax.swing.WindowConstants.EXIT_ON_CLOSE);

        jLabel1.setText("Nombre del libro a agregar");

        jLabel2.setText("ID del autor");

        jButton1.setText("AGREGAR");
        jButton1.addActionListener(this::jButton1ActionPerformed);

        jScrollPane1.setViewportView(jList1);

        jTable1.setModel(new javax.swing.table.DefaultTableModel(
            new Object [][] {
                {},
                {},
                {},
                {}
            },
            new String [] {

            }
        ));
        jScrollPane2.setViewportView(jTable1);

        jTable2.setModel(new javax.swing.table.DefaultTableModel(
            new Object [][] {
                {},
                {},
                {},
                {}
            },
            new String [] {

            }
        ));
        jScrollPane3.setViewportView(jTable2);

        jLabel3.setText("AÑADIR AUTOR");

        jLabel4.setText("AÑADIR LIBRO");

        jLabel5.setText("Nombre del Autor");

        jButton2.setText("AGREGAR");
        jButton2.addActionListener(this::jButton2ActionPerformed);

        jButton3.setText("jButton3");
        jButton3.addActionListener(this::jButton3ActionPerformed);

        javax.swing.GroupLayout layout = new javax.swing.GroupLayout(getContentPane());
        getContentPane().setLayout(layout);
        layout.setHorizontalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(layout.createSequentialGroup()
                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                    .addGroup(layout.createSequentialGroup()
                        .addGap(27, 27, 27)
                        .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING, false)
                            .addGroup(layout.createSequentialGroup()
                                .addComponent(jLabel1, javax.swing.GroupLayout.PREFERRED_SIZE, 157, javax.swing.GroupLayout.PREFERRED_SIZE)
                                .addGap(38, 38, 38)
                                .addComponent(jTextField1, javax.swing.GroupLayout.PREFERRED_SIZE, 98, javax.swing.GroupLayout.PREFERRED_SIZE))
                            .addGroup(javax.swing.GroupLayout.Alignment.TRAILING, layout.createSequentialGroup()
                                .addComponent(jLabel2, javax.swing.GroupLayout.PREFERRED_SIZE, 146, javax.swing.GroupLayout.PREFERRED_SIZE)
                                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED, javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE)
                                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                                    .addGroup(layout.createSequentialGroup()
                                        .addGap(6, 6, 6)
                                        .addComponent(jButton1))
                                    .addComponent(jTextField2, javax.swing.GroupLayout.PREFERRED_SIZE, 98, javax.swing.GroupLayout.PREFERRED_SIZE))))
                        .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                            .addGroup(layout.createSequentialGroup()
                                .addGap(83, 83, 83)
                                .addComponent(jLabel5, javax.swing.GroupLayout.PREFERRED_SIZE, 147, javax.swing.GroupLayout.PREFERRED_SIZE)
                                .addGap(28, 28, 28)
                                .addComponent(jTextField3, javax.swing.GroupLayout.PREFERRED_SIZE, 109, javax.swing.GroupLayout.PREFERRED_SIZE))
                            .addGroup(layout.createSequentialGroup()
                                .addGap(212, 212, 212)
                                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.TRAILING)
                                    .addComponent(jButton3)
                                    .addComponent(jButton2)))))
                    .addGroup(layout.createSequentialGroup()
                        .addGap(106, 106, 106)
                        .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                            .addComponent(jScrollPane2, javax.swing.GroupLayout.PREFERRED_SIZE, 375, javax.swing.GroupLayout.PREFERRED_SIZE)
                            .addComponent(jScrollPane1, javax.swing.GroupLayout.PREFERRED_SIZE, 316, javax.swing.GroupLayout.PREFERRED_SIZE)
                            .addComponent(jScrollPane3, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE)))
                    .addGroup(layout.createSequentialGroup()
                        .addGap(144, 144, 144)
                        .addComponent(jLabel4, javax.swing.GroupLayout.PREFERRED_SIZE, 162, javax.swing.GroupLayout.PREFERRED_SIZE)
                        .addGap(181, 181, 181)
                        .addComponent(jLabel3, javax.swing.GroupLayout.PREFERRED_SIZE, 187, javax.swing.GroupLayout.PREFERRED_SIZE)))
                .addContainerGap(196, Short.MAX_VALUE))
        );
        layout.setVerticalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(layout.createSequentialGroup()
                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                    .addGroup(layout.createSequentialGroup()
                        .addComponent(jLabel4)
                        .addGap(18, 18, 18))
                    .addGroup(javax.swing.GroupLayout.Alignment.TRAILING, layout.createSequentialGroup()
                        .addComponent(jLabel3)
                        .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.UNRELATED)))
                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                    .addComponent(jLabel1)
                    .addComponent(jTextField1, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE)
                    .addComponent(jLabel5)
                    .addComponent(jTextField3, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE))
                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)
                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                    .addComponent(jLabel2)
                    .addComponent(jTextField2, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE)
                    .addComponent(jButton2))
                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)
                .addComponent(jButton1)
                .addGap(9, 9, 9)
                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.TRAILING)
                    .addComponent(jScrollPane1, javax.swing.GroupLayout.PREFERRED_SIZE, 69, javax.swing.GroupLayout.PREFERRED_SIZE)
                    .addComponent(jButton3))
                .addGap(27, 27, 27)
                .addComponent(jScrollPane2, javax.swing.GroupLayout.PREFERRED_SIZE, 113, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addGap(34, 34, 34)
                .addComponent(jScrollPane3, javax.swing.GroupLayout.PREFERRED_SIZE, 129, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addContainerGap(153, Short.MAX_VALUE))
        );

        pack();
    }// </editor-fold>                        

    private void jButton1ActionPerformed(java.awt.event.ActionEvent evt) {                                         
        // TODO add your handling code here:
        añadirLibro();
    }                                        

    private void jButton2ActionPerformed(java.awt.event.ActionEvent evt) {                                         
        // TODO add your handling code here:
        añadirAutor();
    }                                        

    private void jButton3ActionPerformed(java.awt.event.ActionEvent evt) {                                         
        // TODO add your handling code here:
        conectarse();
        llenarTabla();
        llenarTablaAutores();
    }                                        

    /**
     * @param args the command line arguments
     */
    public static void main(String args[]) {
        /* Set the Nimbus look and feel */
        //<editor-fold defaultstate="collapsed" desc=" Look and feel setting code (optional) ">
        /* If Nimbus (introduced in Java SE 6) is not available, stay with the default look and feel.
         * For details see http://download.oracle.com/javase/tutorial/uiswing/lookandfeel/plaf.html 
         */
        try {
            for (javax.swing.UIManager.LookAndFeelInfo info : javax.swing.UIManager.getInstalledLookAndFeels()) {
                if ("Nimbus".equals(info.getName())) {
                    javax.swing.UIManager.setLookAndFeel(info.getClassName());
                    break;
                }
            }
        } catch (ReflectiveOperationException | javax.swing.UnsupportedLookAndFeelException ex) {
            logger.log(java.util.logging.Level.SEVERE, null, ex);
        }
        //</editor-fold>

        /* Create and display the form */
        java.awt.EventQueue.invokeLater(() -> new Interf().setVisible(true));
    }

    // Variables declaration - do not modify                     
    private javax.swing.JButton jButton1;
    private javax.swing.JButton jButton2;
    private javax.swing.JButton jButton3;
    private javax.swing.JLabel jLabel1;
    private javax.swing.JLabel jLabel2;
    private javax.swing.JLabel jLabel3;
    private javax.swing.JLabel jLabel4;
    private javax.swing.JLabel jLabel5;
    private javax.swing.JList<String> jList1;
    private javax.swing.JScrollPane jScrollPane1;
    private javax.swing.JScrollPane jScrollPane2;
    private javax.swing.JScrollPane jScrollPane3;
    private javax.swing.JTable jTable1;
    private javax.swing.JTable jTable2;
    private javax.swing.JTextField jTextField1;
    private javax.swing.JTextField jTextField2;
    private javax.swing.JTextField jTextField3;
    // End of variables declaration                   
}



CREATE TABLE autores (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE libros (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(150) NOT NULL,
    autor_id INT,
    FOREIGN KEY (autor_id) REFERENCES autores(id) ON DELETE CASCADE ON UPDATE CASCADE
);




