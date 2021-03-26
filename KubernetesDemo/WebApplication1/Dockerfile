FROM mcr.microsoft.com/dotnet/aspnet:3.1 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:3.1 AS build
WORKDIR /src
COPY ["KubernetesDemo/WebApplication1/WebApplication1.csproj", "KubernetesDemo/WebApplication1/"]
RUN dotnet restore "KubernetesDemo/WebApplication1/WebApplication1.csproj"
COPY . .
WORKDIR "/src/KubernetesDemo/WebApplication1"
RUN dotnet build "WebApplication1.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "WebApplication1.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "WebApplication1.dll"]
