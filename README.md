# IBM/Laboratoria: Análise de Cancelamentos de Empresa de Telecomunicação
## Introdução ao projeto
Para este projeto, deveríamos organizar e limpar um banco de dados com tabelas múltiplas e identificar os grupos de clientes com maior risco de deixar a empresa.

Os principais objetivos de aprendizado eram:
- Organizar e manipular dados usando os principais comandos do SQL, como SELECT, FROM, WHERE, GROUP BY, ORDER BY, AVG, COUNT. Unir múltiplas tabelas com JOIN. Limpar, filtrar e resumir os dados de acordo com a necessidade do negócio.
- Tomar decisões de negócios com base em dados: resumir e organizar os dados de forma a encontrar informações importantes para apoiar uma decisão de negócios. 
- Visualizar dados em uma ferramenta de Business Intelligence (BI): conhecer o ambiente de desenho do PowerBI, mostrar os principais indicadores e as estruturas em um dashboard que permite ao negócio obter conclusões acionáveis.
- Organizar e comunicar as descobertas: estruturar a análise em um relatório claro e organizado.

## Exploração de dados inicial
#### Unir tabelas usando JOIN
```
CREATE TABLE `telco-353222.Dataset_telco.master_churn`  AS (
SELECT a.* EXCEPT(Zip_Code),
        b.Gender,b.Age,b.Under_30,b.Senior_Citizen,
        b.Married,b.Dependents,b.Number_of_Dependents,
        c.Quarter,c.Referred_a_Friend,c.Number_of_Referrals,
        c.Tenure_in_Months,c.Offer,c.Phone_Service,
        c.Avg_Monthly_Long_Distance_Charges,c.Multiple_Lines,
        c.Internet_Service,c.Internet_Type,c.Avg_Monthly_GB_Download,
        c.Online_Security,c.Online_Backup,c.Device_Protection_Plan,
        c.Premium_Tech_Support,c.Streaming_TV,c.Streaming_Movies,
        c.Streaming_Music,c.Unlimited_Data,c.Contract,c.Paperless_Billing,
        c.Payment_Method,c.Monthly_Charge,c.Total_Charges,c.Total_Refunds,
        c.Total_Extra_Data_Charges,c.Total_Long_Distance_Charges,c.Total_Revenue,
        d.Customer_Status,d.Churn_Label,d.Churn_Value,
        d.Churn_Category,d.Churn_Reason 
FROM `telco-353222.Dataset_telco.churn_location` a
LEFT JOIN `telco-353222.Dataset_telco.churn_demographics` b
ON a.Customer_ID=b.Customer_ID
LEFT JOIN `telco-353222.Dataset_telco.churn_services` c
ON b.Customer_ID=c.Customer_ID
LEFT JOIN `telco-353222.Dataset_telco.churn_status` d
ON c.Customer_ID=d.Customer_ID
)
```
![Captura de Tela (160)](https://user-images.githubusercontent.com/107365734/175069121-37108452-570e-424d-be05-8e1ecf31c072.png)

#### Limpar tabela
```
CREATE OR REPLACE TABLE `projeto.dataset.tabela` AS
(
SELECT * EXCEPT(gender),
CASE
WHEN gender = 'M' THEN 'Male'
WHEN gender = 'F' THEN 'Female'
ELSE gender END as gender
FROM `telco-353222.Dataset_telco.master_churn`
WHERE Age<=80
AND Monthly_Charge > 0
)
```
## Exploração de dados inicial
#### Segmentação de dados por idade
```
WITH tabela AS (
 SELECT *,
    CASE 
    WHEN Age < 41 THEN '1. 0 a 40 anos'
    WHEN Age BETWEEN 41 AND 60 THEN '2. 41 a 60 anos' 
    ELSE '3. Más de 60 años' 
    END AS range_age
    FROM `telco-353222.Dataset_telco.master_churn`
)
SELECT range_age, COUNT(DISTINCT Customer_ID) AS total_clientes
FROM tabela 
GROUP BY range_age
```
#### Segmentação de dados por clientes e referências
```
SELECT City,
    Married,
    reference_range,
    COUNT(DISTINCT Customer_ID) AS total_clientes ,
    SUM(Number_of_Referrals) reference_total
FROM `telco-353222.Dataset_telco.master_churn`
GROUP BY 1,2,3
```
## Definição de churn por segmentos
```
CREATE OR REPLACE  TABLE `projeto.dataset.tabela` AS (
SELECT
  Customer_ID,
  Contract,
  Churn_Value,
  Total_Revenue,
  CASE
    WHEN Contract = 'Month-to-Month' AND Age > 64 THEN 'G1'
    WHEN Contract = 'Month-to-Month' AND Age < 64 AND Number_of_Referrals <= 1 THEN 'G2'
    WHEN Contract != 'Month-to-Month' AND Age > 64 AND Number_of_Referrals <= 1 THEN 'G3'
    WHEN Contract != 'Month-to-Month' AND Tenure_in_Months < 40 THEN 'G4'
    ELSE 'sem grupo'
END AS risk_group

FROM `telco-353222.Dataset_telco.master_churn`
)
```
## Definição do valor dos clientes
```
AS(
WITH base_tenure_prom AS (
        SELECT Contract,AVG(Tenure_in_Months) AS media_tenure
        FROM `telco-353222.Dataset_telco.master_churn`
        GROUP BY 1
)
SELECT a.*,
        b.media_tenure
FROM `telco-353222.Dataset_telco.master_churn` a
LEFT JOIN base_tenure_prom b
ON a.Contract = b.Contract
```
