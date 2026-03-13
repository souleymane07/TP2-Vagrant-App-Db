# TP2 Vagrant : Déploiement d'une application Web Java avec MySQL

## Objectif
Créer deux machines virtuelles avec Vagrant :
- **srv-app** : Serveur d'application (Tomcat + application Java)
- **srv-db** : Serveur de base de données (MySQL)

---

## 1. Création des VM avec Vagrant

### Vagrantfile
```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.define "srv-app" do |app|
    app.vm.hostname = "srv-app"
    app.vm.network "private_network", ip: "192.168.33.10"
    app.vm.network "forwarded_port", guest: 8080, host: 8080
    app.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = "2"
      vb.name = "srv-app"
    end
  end

  config.vm.define "srv-db" do |db|
    db.vm.hostname = "srv-db"
    db.vm.network "private_network", ip: "192.168.33.11"
    db.vm.network "forwarded_port", guest: 3306, host: 3307
    db.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = "1"
      vb.name = "srv-db"
    end
  end
end
```
Commandes
vagrant up
vagrant status

2. Configuration de srv-app
vagrant ssh srv-app

2.1 Installation des JDK
sudo apt update
sudo apt install -y openjdk-8-jdk openjdk-11-jdk openjdk-17-jdk
java -version

2.2 Installation de Tomcat 9
sudo apt install -y tomcat9 tomcat9-admin
sudo systemctl start tomcat9
sudo systemctl enable tomcat9
Configuration utilisateur (/etc/tomcat9/tomcat-users.xml) :
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<user username="admin" password="password" roles="manager-gui,admin-gui"/>
sudo systemctl restart tomcat9

2.3 Création de l'application
cd /tmp
mkdir student-crud && cd student-crud
mkdir -p WEB-INF/classes

index.jsp
```
<!DOCTYPE html>
<html>
<head>
    <title>Gestion des Étudiants</title>
    <style>
        body { font-family: Arial; margin: 20px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #4CAF50; color: white; }
        .button { background-color: #4CAF50; color: white; padding: 10px 15px; text-decoration: none; display: inline-block; margin: 5px; }
        .delete { background-color: #f44336; }
        .edit { background-color: #008CBA; }
    </style>
</head>
<body>
    <h1>Gestion des Étudiants</h1>
    <a href="add.jsp" class="button">Ajouter un étudiant</a>
    
    <%@ page import="java.sql.*" %>
    <%
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;
        
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection(
                "jdbc:mysql://192.168.33.11:3306/etudiant_db", 
                "etudiant_user", 
                "password"
            );
            
            stmt = conn.createStatement();
            rs = stmt.executeQuery("SELECT * FROM etudiants ORDER BY nom");
    %>
    
    <table>
        <tr>
            <th>ID</th><th>Nom</th><th>Prénom</th><th>Email</th><th>Filière</th><th>Actions</th>
        </tr>
        <% while(rs.next()) { %>
        <tr>
            <td><%= rs.getInt("id") %></td>
            <td><%= rs.getString("nom") %></td>
            <td><%= rs.getString("prenom") %></td>
            <td><%= rs.getString("email") %></td>
            <td><%= rs.getString("filiere") %></td>
            <td>
                <a href="edit.jsp?id=<%= rs.getInt("id") %>" class="button edit">Modifier</a>
                <a href="delete.jsp?id=<%= rs.getInt("id") %>" class="button delete" onclick="return confirm('Supprimer ?')">Supprimer</a>
            </td>
        </tr>
        <% } %>
    </table>
    <%
        } catch (Exception e) {
            out.println("<p style='color:red'>Erreur: " + e.getMessage() + "</p>");
        } finally {
            if (rs != null) try { rs.close(); } catch(Exception e) {}
            if (stmt != null) try { stmt.close(); } catch(Exception e) {}
            if (conn != null) try { conn.close(); } catch(Exception e) {}
        }
    %>
</body>
</html>
```
add.jsp
```
<!DOCTYPE html>
<html>
<head>
    <title>Ajouter un étudiant</title>
    <style>
        body { font-family: Arial; margin: 20px; }
        input[type=text], input[type=email] { width: 100%; padding: 12px; margin: 8px 0; }
        input[type=submit] { background-color: #4CAF50; color: white; padding: 14px 20px; border: none; cursor: pointer; }
        .back { background-color: #555; color: white; padding: 14px 20px; text-decoration: none; display: inline-block; }
    </style>
</head>
<body>
    <h1>Ajouter un étudiant</h1>
    <form action="add_action.jsp" method="post">
        <label>Nom:</label> <input type="text" name="nom" required>
        <label>Prénom:</label> <input type="text" name="prenom" required>
        <label>Email:</label> <input type="email" name="email" required>
        <label>Filière:</label> <input type="text" name="filiere" required>
        <input type="submit" value="Ajouter">
        <a href="index.jsp" class="back">Retour</a>
    </form>
</body>
</html>
```
add_action.jsp
```
<%@ page import="java.sql.*" %>
<%
    String nom = request.getParameter("nom");
    String prenom = request.getParameter("prenom");
    String email = request.getParameter("email");
    String filiere = request.getParameter("filiere");
    
    Connection conn = null;
    PreparedStatement pstmt = null;
    
    try {
        Class.forName("com.mysql.cj.jdbc.Driver");
        conn = DriverManager.getConnection(
            "jdbc:mysql://192.168.33.11:3306/etudiant_db", 
            "etudiant_user", 
            "password"
        );
        
        String sql = "INSERT INTO etudiants (nom, prenom, email, filiere) VALUES (?, ?, ?, ?)";
        pstmt = conn.prepareStatement(sql);
        pstmt.setString(1, nom);
        pstmt.setString(2, prenom);
        pstmt.setString(3, email);
        pstmt.setString(4, filiere);
        pstmt.executeUpdate();
        response.sendRedirect("index.jsp");
        
    } catch (Exception e) {
        out.println("Erreur: " + e.getMessage());
    } finally {
        if (pstmt != null) try { pstmt.close(); } catch(Exception e) {}
        if (conn != null) try { conn.close(); } catch(Exception e) {}
    }
%>
```
edit.jsp
```
<!DOCTYPE html>
<html>
<head>
    <title>Modifier un étudiant</title>
    <style>
        body { font-family: Arial; margin: 20px; }
        input[type=text], input[type=email] { width: 100%; padding: 12px; margin: 8px 0; }
        input[type=submit] { background-color: #008CBA; color: white; padding: 14px 20px; border: none; cursor: pointer; }
        .back { background-color: #555; color: white; padding: 14px 20px; text-decoration: none; display: inline-block; }
    </style>
</head>
<body>
    <h1>Modifier un étudiant</h1>
    
    <%@ page import="java.sql.*" %>
    <%
        String id = request.getParameter("id");
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection(
                "jdbc:mysql://192.168.33.11:3306/etudiant_db", 
                "etudiant_user", 
                "password"
            );
            
            pstmt = conn.prepareStatement("SELECT * FROM etudiants WHERE id = ?");
            pstmt.setInt(1, Integer.parseInt(id));
            rs = pstmt.executeQuery();
            
            if (rs.next()) {
    %>
    
    <form action="update.jsp" method="post">
        <input type="hidden" name="id" value="<%= id %>">
        <label>Nom:</label> <input type="text" name="nom" value="<%= rs.getString("nom") %>" required>
        <label>Prénom:</label> <input type="text" name="prenom" value="<%= rs.getString("prenom") %>" required>
        <label>Email:</label> <input type="email" name="email" value="<%= rs.getString("email") %>" required>
        <label>Filière:</label> <input type="text" name="filiere" value="<%= rs.getString("filiere") %>" required>
        <input type="submit" value="Modifier">
        <a href="index.jsp" class="back">Retour</a>
    </form>
    
    <%
            }
        } catch (Exception e) {
            out.println("Erreur: " + e.getMessage());
        } finally {
            if (rs != null) try { rs.close(); } catch(Exception e) {}
            if (pstmt != null) try { pstmt.close(); } catch(Exception e) {}
            if (conn != null) try { conn.close(); } catch(Exception e) {}
        }
    %>
</body>
</html>
```
update.jsp
```
<%@ page import="java.sql.*" %>
<%
    String id = request.getParameter("id");
    String nom = request.getParameter("nom");
    String prenom = request.getParameter("prenom");
    String email = request.getParameter("email");
    String filiere = request.getParameter("filiere");
    
    Connection conn = null;
    PreparedStatement pstmt = null;
    
    try {
        Class.forName("com.mysql.cj.jdbc.Driver");
        conn = DriverManager.getConnection(
            "jdbc:mysql://192.168.33.11:3306/etudiant_db", 
            "etudiant_user", 
            "password"
        );
        
        String sql = "UPDATE etudiants SET nom=?, prenom=?, email=?, filiere=? WHERE id=?";
        pstmt = conn.prepareStatement(sql);
        pstmt.setString(1, nom);
        pstmt.setString(2, prenom);
        pstmt.setString(3, email);
        pstmt.setString(4, filiere);
        pstmt.setInt(5, Integer.parseInt(id));
        pstmt.executeUpdate();
        response.sendRedirect("index.jsp");
        
    } catch (Exception e) {
        out.println("Erreur: " + e.getMessage());
    } finally {
        if (pstmt != null) try { pstmt.close(); } catch(Exception e) {}
        if (conn != null) try { conn.close(); } catch(Exception e) {}
    }
%>
```
delete.jsp
```
<%@ page import="java.sql.*" %>
<%
    String id = request.getParameter("id");
    
    Connection conn = null;
    PreparedStatement pstmt = null;
    
    try {
        Class.forName("com.mysql.cj.jdbc.Driver");
        conn = DriverManager.getConnection(
            "jdbc:mysql://192.168.33.11:3306/etudiant_db", 
            "etudiant_user", 
            "password"
        );
        
        pstmt = conn.prepareStatement("DELETE FROM etudiants WHERE id=?");
        pstmt.setInt(1, Integer.parseInt(id));
        pstmt.executeUpdate();
        response.sendRedirect("index.jsp");
        
    } catch (Exception e) {
        out.println("Erreur: " + e.getMessage());
    } finally {
        if (pstmt != null) try { pstmt.close(); } catch(Exception e) {}
        if (conn != null) try { conn.close(); } catch(Exception e) {}
    }
%>
```
WEB-INF/web.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <display-name>Gestion Étudiants</display-name>
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

2.4 Déploiement
jar -cvf student.war *
sudo cp student.war /var/lib/tomcat9/webapps/

2.5 Installation du connecteur MySQL
cd /tmp
wget https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/8.0.33/mysql-connector-j-8.0.33.jar -O mysql-connector-java-8.0.33.jar
sudo cp mysql-connector-java-8.0.33.jar /usr/share/tomcat9/lib/
sudo systemctl restart tomcat9

3. Configuration de srv-db
vagrant ssh srv-db

3.1 Installation MySQL
sudo apt update
sudo apt install -y mysql-server
sudo systemctl start mysql
sudo systemctl enable mysql

3.2 Création base et utilisateur
sudo mysql
CREATE DATABASE etudiant_db;
CREATE USER 'etudiant_user'@'192.168.33.10' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON etudiant_db.* TO 'etudiant_user'@'192.168.33.10';
CREATE USER 'etudiant_user'@'192.168.33.%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON etudiant_db.* TO 'etudiant_user'@'192.168.33.%';
FLUSH PRIVILEGES;
EXIT;

3.3 Configuration connexions distantes
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
(Modifier : bind-address = 0.0.0.0)
sudo systemctl restart mysql

4. Tests

4.1 Depuis srv-app
sudo apt install -y mysql-client
mysql -h 192.168.33.11 -u etudiant_user -p -e "SHOW DATABASES;"
(Mot de passe: password)

4.2 Création table
mysql -h 192.168.33.11 -u etudiant_user -p etudiant_db -e "
CREATE TABLE IF NOT EXISTS etudiants (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nom VARCHAR(50) NOT NULL,
    prenom VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    filiere VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);"

4.3 Insertion test
mysql -h 192.168.33.11 -u etudiant_user -p etudiant_db -e "
INSERT INTO etudiants (nom, prenom, email, filiere) VALUES 
('Dupont', 'Jean', 'jean.dupont@email.com', 'Informatique');"
mysql -h 192.168.33.11 -u etudiant_user -p etudiant_db -e "SELECT * FROM etudiants;"

4.4 Test navigateur
URL : http://localhost:8080/student/
