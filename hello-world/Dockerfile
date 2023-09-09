FROM registry.suse.com/bci/dotnet-sdk:7.0 as build-env
WORKDIR /src

COPY . ./
RUN dotnet restore

RUN dotnet publish -c release -o out

FROM registry.suse.com/bci/dotnet-aspnet:7.0
WORKDIR /app
COPY --from=build-env /src/out .
EXPOSE 80
ENTRYPOINT [ "dotnet", "hello-world.dll" ]