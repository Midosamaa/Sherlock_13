# Makefile pour le projet Sherlock 13

CC = gcc
CFLAGS = -I/usr/include/SDL2 -g
LDFLAGS = -lSDL2_image -lSDL2_ttf -lSDL2 -lpthread

CLIENT_SRC = sh13.c
CLIENT_BIN = sh13

SERVER_SRC = server.c
SERVER_BIN = server

all: $(CLIENT_BIN) $(SERVER_BIN)

$(CLIENT_BIN): $(CLIENT_SRC)
	$(CC) -o $(CLIENT_BIN) $(CFLAGS) $(CLIENT_SRC) $(LDFLAGS)

$(SERVER_BIN): $(SERVER_SRC)
	$(CC) -o $(SERVER_BIN) $(SERVER_SRC)

clean:
	rm -f $(CLIENT_BIN) $(SERVER_BIN)
