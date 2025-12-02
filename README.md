import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

// زر دائري مع حركة ثلاثية الأبعاد وظل
class RoundButton3D extends JButton {
    private Color baseColor;
    private int pressedOffset = 2;

    public RoundButton3D(String text, Color base) {
        super(text);
        baseColor = base;
        setFont(new Font("SansSerif", Font.BOLD, 24));
        setFocusPainted(false);
        setBorderPainted(false);
        setContentAreaFilled(false);
        setForeground(Color.WHITE);
        setOpaque(false);
        setPreferredSize(new Dimension(70,70));

        // تأثير الضغط ثلاثي الأبعاد
        getModel().addChangeListener(e -> repaint());
    }

    @Override
    protected void paintComponent(Graphics g) {
        Graphics2D g2 = (Graphics2D) g.create();
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);

        int offset = getModel().isPressed() ? pressedOffset : 0;

        // تدرج اللون
        Color top = baseColor.brighter();
        Color bottom = baseColor.darker();
        GradientPaint gp = new GradientPaint(0,0+offset,top,0,getHeight()+offset,bottom);
        g2.setPaint(gp);
        g2.fillOval(0,0,getWidth(),getHeight());

        // ظل عند الضغط
        if (getModel().isPressed()) {
            g2.setColor(new Color(0,0,0,50));
            g2.fillOval(0,0,getWidth(),getHeight());
        }

        super.paintComponent(g2);
        g2.dispose();
    }

    @Override
    protected void paintBorder(Graphics g) {
        g.setColor(baseColor.darker().darker());
        g.drawOval(0,0,getWidth()-1,getHeight()-1);
    }

    @Override
    public boolean contains(int x, int y) {
        int radius = Math.min(getWidth(), getHeight())/2;
        int centerX = getWidth()/2;
        int centerY = getHeight()/2;
        return (x-centerX)*(x-centerX) + (y-centerY)*(y-centerY) <= radius*radius;
    }
}

public class IPhoneCalculator3D extends JFrame implements ActionListener {
    private JTextField display;
    private String currentOp = "";
    private double storedValue = 0;
    private boolean startNewNumber = true;

    public IPhoneCalculator3D() {
        setTitle("iPhone Calculator 3D");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(360, 520);
        setLocationRelativeTo(null);
        setLayout(new BorderLayout(6,6));

        display = new JTextField("0");
        display.setFont(new Font("SansSerif", Font.PLAIN, 36));
        display.setHorizontalAlignment(JTextField.RIGHT);
        display.setEditable(false);
        display.setBackground(Color.BLACK);
        display.setForeground(Color.WHITE);
        display.setBorder(BorderFactory.createEmptyBorder(10,10,10,10));
        add(display, BorderLayout.NORTH);

        String[] buttons = {
            "C","±","%","/",
            "7","8","9","*",
            "4","5","6","-",
            "1","2","3","+",
            "0",".","←","="
        };

        JPanel panel = new JPanel();
        panel.setLayout(new GridLayout(5,4,12,12));
        panel.setBackground(Color.BLACK);

        for (String text : buttons) {
            Color color;
            switch(text) {
                case "C": case "±": case "%": case "←":
                    color = new Color(200,200,200);
                    break;
                case "/": case "*": case "-": case "+": case "=":
                    color = new Color(255,165,0);
                    break;
                default:
                    color = new Color(50,50,50);
                    break;
            }
            RoundButton3D btn = new RoundButton3D(text, color);
            btn.addActionListener(this);
            panel.add(btn);
        }

        add(panel, BorderLayout.CENTER);
        setVisible(true);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        JButton btn = (JButton)e.getSource();
        Toolkit.getDefaultToolkit().beep(); // صوت الضغط
        shakeButton(btn); // تأثير هز خفيف

        String cmd = btn.getText();

        if ("0123456789".contains(cmd)) numberPressed(cmd);
        else if (cmd.equals(".")) dotPressed();
        else if (cmd.equals("C")) clearAll();
        else if (cmd.equals("←")) backspace();
        else if (cmd.equals("±")) toggleSign();
        else if (cmd.equals("%")) percent();
        else if (cmd.equals("=")) equals();
        else operatorPressed(cmd);
    }

    private void shakeButton(JButton btn) {
        Point p = btn.getLocation();
        int x = p.x;
        int y = p.y;
        for(int i=0;i<2;i++){
            btn.setLocation(x+2, y);
            try { Thread.sleep(20); } catch(Exception ex){}
            btn.setLocation(x-2, y);
            try { Thread.sleep(20); } catch(Exception ex){}
        }
        btn.setLocation(x, y);
    }

    private void numberPressed(String num) {
        if (startNewNumber) { display.setText(num); startNewNumber = false; }
        else display.setText(display.getText().equals("0") ? num : display.getText() + num);
    }

    private void dotPressed() {
        if (startNewNumber) { display.setText("0."); startNewNumber = false; }
        else if (!display.getText().contains(".")) display.setText(display.getText() + ".");
    }

    private void clearAll() {
        display.setText("0");
        storedValue = 0;
        currentOp = "";
        startNewNumber = true;
    }

    private void backspace() {
        if (startNewNumber) return;
        String s = display.getText();
        if (s.length() <= 1) { display.setText("0"); startNewNumber = true; }
        else display.setText(s.substring(0, s.length() - 1));
    }

    private void toggleSign() {
        String s = display.getText();
        if (s.equals("0") || s.equals("0.0")) return;
        display.setText(s.startsWith("-") ? s.substring(1) : "-" + s);
    }

    private void percent() {
        double val = Double.parseDouble(display.getText()) / 100.0;
        display.setText(formatNumber(val));
        startNewNumber = true;
    }

    private void operatorPressed(String op) {
        double displayValue = Double.parseDouble(display.getText());
        if (!currentOp.isEmpty()) storedValue = compute(storedValue, displayValue, currentOp);
        else storedValue = displayValue;
        currentOp = op;
        startNewNumber = true;
        display.setText(formatNumber(storedValue));
    }

    private void equals() {
        if (currentOp.isEmpty()) return;
        double displayValue = Double.parseDouble(display.getText());
        double result = compute(storedValue, displayValue, currentOp);
        display.setText(formatNumber(result));
        currentOp = "";
        storedValue = 0;
        startNewNumber = true;
    }

    private double compute(double a, double b, String op) {
        switch(op) {
            case "+": return a + b;
            case "-": return a - b;
            case "*": return a * b;
            case "/": return b == 0 ? 0 : a / b;
            default: return b;
        }
    }

    private String formatNumber(double v) {
        if (v == (long) v) return String.format("%d", (long)v);
        else return String.valueOf(v);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(IPhoneCalculator3D::new);
    }
}
