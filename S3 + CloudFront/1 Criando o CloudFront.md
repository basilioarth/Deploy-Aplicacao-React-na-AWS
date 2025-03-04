**Resumo da Aula: Configuração de Site Estático com AWS S3, CloudFront e Automação via GitHub Actions**  

---

### **Objetivo Principal**  
Configurar um site estático na AWS usando **S3** para armazenamento e **CloudFront** como CDN (Content Delivery Network), garantindo baixa latência, segurança e entrega global de conteúdo. Além disso, automatizar o processo de deploy com **GitHub Actions**.

---

### **Principais Conceitos e Passos**  

#### **1. Configuração do Amazon S3**  
- **Criação do Bucket**:  
  - Nome único globalmente (ex: `rocketseat-upload-widget-cdn`).  
  - Habilitar **Static Website Hosting** nas propriedades do bucket.  
  - Definir `index.html` como documento padrão.  

- **Permissões**:  
  - Desabilitar **Block Public Access** para permitir acesso público.  
  - Configurar **Bucket Policy** para permitir leitura (`s3:GetObject`):  
    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": "*",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::rocketseat-upload-widget-cdn/*"
        }
      ]
    }
    ```  
  - Associar uma política de acesso (**OAC - Origin Access Control**) para restringir acesso somente via CloudFront.  

- **Upload dos Arquivos**:  
  - Fazer upload dos arquivos estáticos (HTML, CSS, JS, imagens) para o bucket.  

---

#### **2. Configuração do Amazon CloudFront**  
- **Criação da Distribuição**:  
  - Origem: Bucket S3 configurado (`rocketseat-upload-widget-cdn.s3.amazonaws.com`).  
  - Habilitar **HTTPS** para segurança.  
  - Definir **Default Root Object** como `index.html`.  
  - Configurar **Cache Behaviors** para otimizar entrega (ex: TTL padrão de 24 horas).  

- **Segurança**:  
  - Usar **Web Application Firewall (WAF)** para proteção contra ataques (opcional).  
  - Restringir acesso via **Signed URLs/Cookies** (não utilizado neste exemplo).  

- **Global Delivery**:  
  - Selecionar **All Edge Locations** para entrega global com baixa latência.  

---

#### **3. Automação com GitHub Actions**  
- **Workflow (`main.yml`)** para CI/CD:  
  1. **Build da Aplicação**: Gerar arquivos estáticos (ex: `npm run build`).  
  2. **Configurar Credenciais AWS**:  
     ```yaml
     - name: Configure AWS Credentials
       uses: aws-actions/configure-aws-credentials@v1
       with:
         aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
         aws-secret-access-key: ${{ secrets.AWS_SECRET_KEY }}
         aws-region: us-east-1
     ```  
  3. **Upload para o S3**:  
     ```yaml
     - name: Deploy to S3
       run: aws s3 sync ./dist s3://rocketseat-upload-widget-cdn --delete
     ```  
  4. **Invalidar Cache do CloudFront**:  
     ```yaml
     - name: Invalidate CloudFront Cache
       run: aws cloudfront create-invalidation --distribution-id ${{ secrets.CF_DISTRIBUTION_ID }} --paths "/*"
     ```  

---

### **Problemas Comuns e Soluções**  
- **Acesso Negado no S3**:  
  - Verificar **Bucket Policy** e **Block Public Access**.  
  - Garantir que o **OAC** está vinculado ao CloudFront.  
- **Conteúdo Desatualizado no CloudFront**:  
  - Invalidar cache manualmente ou via GitHub Actions após novos deploys.  
- **Latência Alta**:  
  - Usar **Edge Locations** mais próximas do usuário final.  

---

### **Dicas do Professor**  
- **Custos**:  
  - Monitorar uso do S3 e CloudFront para evitar cobranças inesperadas.  
  - Usar **AWS Free Tier** para testes iniciais.  
- **Segurança**:  
  - Habilitar **Versioning** no S3 para recuperação de arquivos.  
  - Usar **WAF** para bloquear tráfego malicioso.  
- **Performance**:  
  - Habilitar compressão (**GZIP/Brotli**) no CloudFront.  

---

### **Conclusão**  
A combinação **S3 + CloudFront** oferece uma solução robusta para hospedar sites estáticos com alta disponibilidade, segurança e performance global. A automação via GitHub Actions elimina etapas manuais, garantindo deploys rápidos e consistentes. Nas próximas aulas, explore a integração com domínios personalizados e monitoramento avançado para um fluxo profissional completo! 🚀