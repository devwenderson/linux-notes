# Código para estudar

### O que estudar:
- Wordpress hooks
- Custom wordpress database tables 
- SQL querys
- Library TCPDF
- Library PHPMailer e WPMAIL 

```php
<?php
/*
 * Plugin Name:       Budget generate
 * Description:       Gera o orçamento de um projeto de placa solares
 * Version:           1.1
 * Author:            Wenderson
 */

if (!defined('ABSPATH')) {
    exit; // Segurança
}

function bill_calculate($bill)
{
    $bill = floatval($bill);
    $media_mensal = $bill * 0.95;
    $media_anual = $media_mensal * 12;
    $qtd_paineis = $media_mensal / ((30 * 0.72 * 4.98 * 560) / 1000);
    $qtd_paineis = round($qtd_paineis);
    $economia_mensal = $bill;
    $economia_anual = $bill * 12;

    return compact('media_mensal', 'media_anual', 'qtd_paineis', 'economia_mensal', 'economia_anual');
};

function project_price_calculate($qtd_paineis)
{
    $project_price = 0;

    if ($qtd_paineis > 0 && $qtd_paineis <= 5) {
        $project_price = $qtd_paineis * 4000;
    } elseif ($qtd_paineis > 5 && $qtd_paineis <= 10) {
        $project_price = $qtd_paineis * 3500;
    } elseif ($qtd_paineis > 10 && $qtd_paineis <= 30) {
        $project_price = $qtd_paineis * 3000;
    } elseif ($qtd_paineis > 30 && $qtd_paineis <= 40) {
        $project_price = $qtd_paineis * 2500;
    } elseif ($qtd_paineis > 30 && $qtd_paineis <= 40) {
        $project_price = $qtd_paineis * 2500;
    }

    return $project_price;
}

function budget_generate_install()
{
    global $wpdb;
    $table_name = $wpdb->prefix . 'tb_budget';
    $charset_collate = $wpdb->get_charset_collate();
    $sql = "CREATE TABLE IF NOT EXISTS $table_name (
        id INT AUTO_INCREMENT PRIMARY KEY,
        media_mensal DECIMAL(10,2) NOT NULL,
        media_anual DECIMAL(10,2) NOT NULL,
        qtd_paineis INT NOT NULL,
        economia_mensal DECIMAL(10,2) NOT NULL,
        economia_anual DECIMAL(10,2) NOT NULL,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    ) $charset_collate;";
    require_once(ABSPATH . 'wp-admin/includes/upgrade.php');
    dbDelta($sql);
}

register_activation_hook(__FILE__, 'budget_generate_install');

function budget_form() {
    ob_start(); ?>
    <form action="" method="POST">
        <input type="text" name="username" id="username" placeholder="Seu nome" required>
        <input type="text" name="email" id="email" placeholder="Seu email" required>
        <input type="text" name="bill" id="bill" placeholder="Sua fatura" required>
        <button type="submit">Gerar orçamento</button>
    </form>
    <?php
    return ob_get_clean();
};

add_shortcode('budget_form', 'budget_form');

function budget_form_process() {
    if ($_SERVER['REQUEST_METHOD'] == 'POST' && isset($_POST['bill'])) {
        require_once(plugin_dir_path(__FILE__) . 'tcpdf/tcpdf.php');

        // DATABASE INSERT
        global $wpdb;
        $username = $_POST['username'];
        $bill_data = $_POST['bill'];
        $bill = bill_calculate($bill_data);
        $price = project_price_calculate($bill['qtd_paineis']);

        // $wpdb->insert($wpdb->prefix . 'tb_budget', [
        //     'media_mensal' => $bill['media_mensal'],
        //     'media_anual' => $bill['media_anual'],
        //     'qtd_paineis' => $bill['qtd_paineis'],
        //     'economia_mensal' => $bill['economia_mensal'],
        //     'economia_anual' => $bill['economia_anual'],
        // ]);

        // PDF GEN
        $pdf = new TCPDF();
        $pdf->AddPage();
        $html = "
            <h1>Cliente: </h1> <span>{$username}</span>
            <h4>Preço:</h4> <span>{$price}</span>
            <h4>Média mensal: </h4> <span>{$bill['media_mensal']}</span>
        ";
        $pdf->writeHTML($html, true, false, true, false, '');

        // SALVA O PDF
        $pdf_output = plugin_dir_path(__FILE__).'temp/doc.pdf';
        $pdf->Output($pdf_output, 'F');

        // PARÂMETROS PARA ENVIAR O EMAIL
        $to = $_POST['email'];;
        $subject = "Teste de envio de email";
        $message = 'Olá, aqui está seu email gerado';

        // CABEÇALHO DO EMAIL
        $headers = array("Content-Type:text/html; charset=UTF-8", "From: wenderson1909@gmail.com");

        // ANEXOS
        $attachments = array(plugin_dir_path(__FILE__).$pdf_output);

        // ENVIA O EMAIL
        $mail_sent = wp_mail($to, $subject, $message, $headers, $pdf_output);

        if ($mail_sent) {
            echo 'E-mail enviado com sucesso!';
        } else {
            echo 'Erro ao enviar e-mail.';
        }
        
        // Limpar o arquivo temporário após o envio
        unlink($pdf_output); // Remove o arquivo PDF após o envio
    }
};

add_action('init', 'budget_form_process');
```