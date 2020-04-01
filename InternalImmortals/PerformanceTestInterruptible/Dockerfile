# TODO: should migrate to depend only on 'ambrosia' not 'ambrosia-dev':
FROM ambrosia/ambrosia-dev

# For using Tmux inside the container:
# ------------------------------------------
RUN apt-get install -y tmux
# RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && locale-gen
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y locales
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_US.UTF-8
# ----------------------------------------

# Publish to a local directory:
RUN echo "&&&&&&&&&&&&&0 ${PATH}:/ambrosia/InternalImmortals/PerformanceTestInterruptible/Client/publish:/ambrosia/InternalImmortals/PerformanceTestInterruptible/Server/publish"
ENV PATH="${PATH}:/ambrosia/InternalImmortals/PerformanceTestInterruptible/Client/publish:/ambrosia/InternalImmortals/PerformanceTestInterruptible/Server/publish"

# ------------------------------------------------------------

# This Dockerfile is an example of adding the application
# *piece-by-piece* to enable incremental rebuilds.
RUN echo "&&&&&&&&&&&&&1 IJob /ambrosia/InternalImmortals/PerformanceTestInterruptible/IJob"
ADD IJob /ambrosia/InternalImmortals/PerformanceTestInterruptible/IJob

RUN echo "&&&&&&&&&&&&&15 WORKDIR /ambrosia/InternalImmortals/PerformanceTestInterruptible"
WORKDIR /ambrosia/InternalImmortals/PerformanceTestInterruptible
RUN echo "&&&&&&&&&&&&&2 dotnet publish -o publish $BLDFLAGS IJob/IJob.csproj"
RUN dotnet publish -o publish $BLDFLAGS IJob/IJob.csproj


# DANGER, WARNING, FIXME: it is UNSAFE to dotnet publish twice to the same absolute path:
RUN echo "&&&&&&&&&&&&&25 API /ambrosia/InternalImmortals/PerformanceTestInterruptible/API"
ADD API /ambrosia/InternalImmortals/PerformanceTestInterruptible/API
RUN echo "&&&&&&&&&&&&&3 dotnet publish -o publish $BLDFLAGS API/ServerAPI.csproj"
RUN dotnet publish -o publish $BLDFLAGS API/ServerAPI.csproj

# Run the code-generation tool:
RUN echo "&&&&&&&&&&&&&4 rm -rf ./GeneratedSourceFiles && mkdir -p GeneratedSourceFiles "
RUN rm -rf ./GeneratedSourceFiles && mkdir -p GeneratedSourceFiles 

RUN echo "&&&&&&&&&&&&&5 AmbrosiaCS CodeGen -a IJob/publish/IJob.dll -a API/publish/ServerAPI.dll -p IJob/IJob.csproj -p API/ServerAPI.csproj -o PTIAmbrosiaGeneratedAPI -f netcoreapp3.1 -f net46"
RUN AmbrosiaCS CodeGen -a IJob/publish/IJob.dll -a API/publish/ServerAPI.dll -p IJob/IJob.csproj -p API/ServerAPI.csproj -o PTIAmbrosiaGeneratedAPI -f netcoreapp3.1 -f net46
#*#*#*# Test statement
#RUN AmbrosiaCS CodeGen -a /IJob/bin/Release/netcoreapp3.1/linux-x64/IJob.dll -a API/bin/Release/netcoreapp3.1/linux-x64/ServerAPI.dll -p IJob/IJob.csproj -p API/ServerAPI.csproj -o PTIAmbrosiaGeneratedAPI -f netcoreapp3.1 -f net46
#*#*#*#


# Be wary: absolute paths have rather different behavior for dotnet publish.
ENV BUILDIT="dotnet publish -o publish $BLDFLAGS"

# Bulid/ppublish the RPC wrappers whose project we just created:
RUN $BUILDIT GeneratedSourceFiles/PTIAmbrosiaGeneratedAPI/latest/PTIAmbrosiaGeneratedAPI.csproj

# publish the client/server executables that we'll actually run:
ADD Client /ambrosia/InternalImmortals/PerformanceTestInterruptible/Client
ADD Server /ambrosia/InternalImmortals/PerformanceTestInterruptible/Server
RUN $BUILDIT Client/Job.csproj
RUN $BUILDIT Server/Server.csproj

# Add the scripts for invoking PTI inside the container:
ADD run_*.sh                /ambrosia/InternalImmortals/PerformanceTestInterruptible/
ADD default_var_settings.sh /ambrosia/InternalImmortals/PerformanceTestInterruptible/

env PATH="$PATH:/ambrosia/InternalImmortals/PerformanceTestInterruptible/bin"

# ImmortalCoordinator default port:
EXPOSE 1500
# Trying this secondary port as well:
EXPOSE 50500
