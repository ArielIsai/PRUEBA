# PRUEBA
/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 * Click nbfs://nbhost/SystemFileSystem/Templates/Classes/Class.java to edit this template
 */
package prueba2.pkg3;

import java.awt.Color;
import java.awt.Component;
import javax.swing.table.DefaultTableModel;
import java.sql.*;
import javax.swing.JOptionPane;
import javax.swing.JTable;
import javax.swing.table.TableCellRenderer;

/**
 *
 * @author Ariel
 */
public class tablaColores2 extends JTable{
   
    public void llenar(String bd, String tabla, String columna){
        DefaultTableModel m = new DefaultTableModel();
        try{
            Connection c = Prueba23.conexion(bd);
            String sql = "SELECT "+columna+" FROM "+tabla;
            Statement s = c.createStatement();
            ResultSet rs = s.executeQuery(sql);
            m.addColumn(columna);
            
            while(rs.next()){
                m.addRow(new Object[]{rs.getObject(columna)});
            }
            this.setModel(m);
        }catch(Exception e){
            JOptionPane.showMessageDialog(null, "Error: "+e);
        }
    }

        @Override
    public Component prepareRenderer(TableCellRenderer r, int fila, int col) {

        Component c = super.prepareRenderer(r, fila, col);
        Object v = getValueAt(fila, col);

        if (v != null) {
            try {
                int n = Integer.parseInt(v.toString());

                if (n % 2 == 0) {
                    c.setBackground(Color.CYAN);   // par
                } else {
                    c.setBackground(Color.PINK);   // impar
                }

            } catch (NumberFormatException e) {
                c.setBackground(Color.GREEN); // texto
            }
        }

        if (isCellSelected(fila, col)) {
            c.setBackground(getSelectionBackground());
        }

        return c;
    }
        
    }


    OTRA CLASE
   public static Connection conexion(String bd){
        Connection c = null;
        try{
            Class.forName("com.mysql.cj.jdbc.Driver");
            String url = "jdbc:mysql://localhost:3306/"+bd;
            c = DriverManager.getConnection(url, "root", "");
            JOptionPane.showMessageDialog(null, "conect");
        }catch(Exception e){
            JOptionPane.showMessageDialog(null, "Error: "+e);
        }
        return c;
    }

   GUI
   public Interfaz() {
        initComponents();
        
        llenarValores();
    }

    public void llenarValores(){
        tablaColores21.llenar("uta_fisei_eventosconfig", "carrera", "NOMBRE_CARRERA");
        
    }


   COMBO
   public class comboColores2 extends JComboBox<Object>{
    
    public comboColores2(){
        this.setRenderer(new RendererColores());
    }
    
    public void llenarCombo(String bd, String tabla, String columna){
        DefaultComboBoxModel<Object> cb = new DefaultComboBoxModel<>();
        try {
            Connection c = Prueba23.conexion(bd);
            String sql = "SELECT "+columna+" FROM "+tabla;
            Statement s = c.createStatement();
            ResultSet rs;
            rs = s.executeQuery(sql);
            
            while(rs.next()){
                cb.addElement(rs.getObject(columna));
            }
            
            this.setModel(cb);
        } catch (SQLException ex) {
            System.getLogger(comboColores2.class.getName()).log(System.Logger.Level.ERROR, (String) null, ex);
        }
        
    }
    
    
    private class RendererColores extends BasicComboBoxRenderer {

        @Override
        public Component getListCellRendererComponent(
                JList list, Object value, int index,
                boolean isSelected, boolean cellHasFocus) {

            super.getListCellRendererComponent(
                    list, value, index, isSelected, cellHasFocus);

            if (value != null) {
                try {
                    int n = Integer.parseInt(value.toString());

                    if (n % 2 == 0) {
                        setBackground(Color.CYAN);   // par
                    } else {
                        setBackground(Color.PINK);   // impar
                    }

                } catch (NumberFormatException e) {
                    setBackground(Color.LIGHT_GRAY); // texto
                }
            }

            if (isSelected) {
                setBackground(list.getSelectionBackground());
                setForeground(list.getSelectionForeground());
            }

            return this;
        }
    }

   POR SI ACASO
   this.setDefaultRenderer(Object.class, new RendererColores());
   this.setRenderer(new RendererColores());
