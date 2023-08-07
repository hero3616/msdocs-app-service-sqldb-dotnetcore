Azure CLI

az login

$azureaduser=(az ad user list --filter "userPrincipalName eq '0246746@itwconnect.com'" --query '[].id' --output tsv)

echo $azureaduser

az sql server ad-admin create `
    --resource-group "sandbox-cem-rg" `
    --server-name "msdocs-core-sql-tutorial1-server" `
    --display-name ADMIN `
    --object-id $azureaduser

# Name is the Web App (App Service) name
az webapp identity assign `
    --resource-group "sandbox-cem-rg" `
    --name "msdocs-core-sql-tutorial1"

# Will open popup to login
# Instead of sqlcmd you can use SSMS as well
sqlcmd -S "msdocs-core-sql-tutorial1-server.database.windows.net" -d "msdocs-core-sql-tutorial1-database" -U "0246746@itwconnect.com" -G -l 30

# This is similar to adding IIS Application Pool Identity to SQL Server in a VM or local
# Docs: <identity-name> is the name of the managed identity in Azure AD. If the identity is system-assigned, the name is always the same as the name of your App Service app ("msdocs-core-sql-tutorial1").
CREATE USER [msdocs-core-sql-tutorial1] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [msdocs-core-sql-tutorial1];
ALTER ROLE db_datawriter ADD MEMBER [msdocs-core-sql-tutorial1];
ALTER ROLE db_ddladmin ADD MEMBER [msdocs-core-sql-tutorial1];
GO

"exit" to leave sqlcmd

# Now appsettings "Authentication=Active Directory Default;" should work both on local and in App Service, so delete the configuration in App Service
az webapp config connection-string delete `
    --resource-group "sandbox-cem-rg" `
    --name "msdocs-core-sql-tutorial1" `
    --setting-names "AZURE_SQL_CONNECTIONSTRING"

Redis Cache connection
- You must enable non-SSL connections under Settings -> Advanced settings (otherwise you have to use a proxy like stunnel)
- You must enable public network access under Settings -> Private Endpoint
- Both VS Code and redis-cli works

redis-cli -h msdocs-core-sql-tutorial1-cache.redis.cache.windows.net -p 6379 -a B0OBftEV7BiysnaQ00Q4zboR3bdFLiOA2AzCaEKfQd8=
> select 0
> keys *
> flushdb