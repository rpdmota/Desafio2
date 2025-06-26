Visão Geral do Monitoramento Azure
O Azure oferece um conjunto abrangente de ferramentas de monitoramento através do Azure Monitor, que coleta, analisa e age sobre dados de telemetria de seus recursos Azure e ambientes híbridos.

1. Configuração Básica do Azure Monitor
1.1 Habilitar Azure Monitor
Acesse o Portal Azure (portal.azure.com)
Navegue até "Monitor" no menu de serviços
Configure as opções básicas de coleta de dados
1.2 Criar Workspace do Log Analytics
New-AzOperationalInsightsWorkspace -ResourceGroupName "MyResourceGroup" -Name "MyWorkspace" -Location "EastUS"

# Para uma VM existente
2. Configuração
Enable-AzOperationalInsightsVMMonitoring -ResourceGroupName "MyResourceGroup" -Name "MyVM" -WorkspaceId <WorkspaceID> -WorkspaceKey <WorkspaceKey>
2.2 Configurar Coleta de Dados
No portal, navegue até sua VM
Selecione "Diagnostics settings" em "Monitoring"
Adicione configurações para coletar:
Logs do sistema
Métricas de desempenho
Logs de infraestrutura

3. Monitoramento de Aplicações
3.1 Application Insights
New-AzApplicationInsights -ResourceGroupName "MyResourceGroup" -Name "MyAppInsights" -Location "EastUS"
3.2 Configurar Monitoramento para:
Aplicações Web (ASP.NET, Java, Node.js, etc.)
Funções Azure
Aplicativos em contêineres

4. Configuração de Alertas
4.1 Criar Grupo de Ações
$actionGroup = New-AzActionGroup -ResourceGroupName "MyResourceGroup" -Name "CriticalAlertsGroup" -ShortName "CritAlert" -EmailReceiver @{"Name"="Admin";"EmailAddress"="admin@example.com"}
4.2 Criar Regra de Alerta de Métrica
Add-AzMetricAlertRuleV2 -Name "HighCPUAlert" -ResourceGroupName "MyResourceGroup" -WindowSize 00:05:00 -Frequency 00:01:00 -TargetResourceId "/subscriptions/.../resourceGroups/.../providers/Microsoft.Compute/virtualMachines/MyVM" -Condition "avg Percentage CPU > 90" -Severity 2 -ActionGroupId $actionGroup.Id

5. Análise de Logs com Log Analytics
5.1 Consultas Básicas
// Consulta de desempenho
Perf 
| where TimeGenerated > ago(1h) 
| where CounterName == "% Processor Time" 
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| render timechart
5.2 Consultas Avançadas
// Correlacionando eventos de segurança com desempenho
SecurityEvent
| where EventID == 4625 // Logon falho
| join kind=inner (
    Perf 
    | where CounterName == "% Processor Time"
) on Computer
| project TimeGenerated, Computer, EventID, CounterValue, Account

6. Monitoramento de Rede
6.1 NSG Flow Logs
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName "MyResourceGroup" -Name "MyWorkspace"
$storageAccount = Get-AzStorageAccount -ResourceGroupName "MyResourceGroup" -Name "mystorageaccount"
Set-AzNetworkWatcherConfigFlowLog -NetworkWatcherName "NetworkWatcher_eastus" -ResourceGroupName "NetworkWatcherRG" -TargetResourceId "/subscriptions/.../resourceGroups/.../providers/Microsoft.Network/networkSecurityGroups/MyNSG" -StorageAccountId $storageAccount.Id -Enabled $true -Workspace $workspace -FormatType Json -FormatVersion 2

7. Painéis e Visualização
7.1 Criar Painel Personalizado
$dashboard = New-AzPortalDashboard -ResourceGroupName "MyResourceGroup" -Name "MyDashboard" -DashboardPath ".\dashboard.json"
7.2 Widgets Recomendados:
Gráficos de métricas
Resultados de consultas de log
Status de recursos
Mapas de aplicativos

8. Melhores Práticas
Padronização: Use tags consistentes em todos os recursos
Hierarquia: Implemente configurações em nível de assinatura quando possível
Retenção: Configure políticas de retenção adequadas (até 2 anos para logs)
Governança: Utilize Azure Policy para impor configurações de monitoramento
Custos: Monitore o custo do próprio Azure Monitor

9. Monitoramento Híbrido
9.1 Arc-enabled servers
Connect-AzConnectedMachine -ResourceGroupName "MyResourceGroup" -Name "MyOnPremServer" -Location "EastUS"
9.2 Agente do Log Analytics para servidores não-Azure
New-AzConnectedMachineExtension -Name "OMSExtension" -ResourceGroupName "MyResourceGroup" -MachineName "MyOnPremServer" -Location "EastUS" -Publisher "Microsoft.EnterpriseCloud.Monitoring" -ExtensionType "OMSExtension" -TypeHandlerVersion "1.13" -Settings @{"workspaceId" = "<WorkspaceID>"} -ProtectedSettings @{"workspaceKey" = "<WorkspaceKey>"}

10. Automação de Respostas
10.1 Runbooks de Automação
$automationAccount = Get-AzAutomationAccount -ResourceGroupName "MyResourceGroup" -Name "MyAutomationAccount"
$runbook = Import-AzAutomationRunbook -Path ".\MyRunbook.ps1" -Name "ScaleDownVMs" -Type PowerShell -AutomationAccountName $automationAccount.AutomationAccountName -ResourceGroupName $automationAccount.ResourceGroupName -Published
10.2 Alertas com Ações Automatizadas
$webhookReceiver = New-AzActionGroupReceiver -Name "AutomationWebhook" -WebhookReceiver -ServiceUri "https://s1events.azure-automation.net/webhooks?token=..."
Set-AzActionGroup -ResourceGroupName "MyResourceGroup" -Name "MyActionGroup" -ShortName "MyActGrp" -Receiver $webhookReceiver

