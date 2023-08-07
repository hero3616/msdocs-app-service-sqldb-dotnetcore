Azure CLI

az login

$azureaduser=(az ad user list --filter "userPrincipalName eq '0246746@itwconnect.com'" --query '[].id' --output tsv)

echo $azureaduser

az sql server ad-admin create `
    --resource-group "sandbox-cem-rg" `
    --server-name "msdocs-core-sql-tutorial1-server" `
    --display-name ADMIN `
    --object-id $azureaduser
