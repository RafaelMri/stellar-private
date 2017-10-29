Horizon installation
--------------------

    Download pre-built binary from GitHub: https://github.com/stellar/horizon/releases
    Unzip
    export DATABASE_URL=postgres_end_point
    horizon db init
    Put file:
    	Horizon service file
    Copy horizon service file to /etc/systemd/system/horizon.service
    Start service: sudo systemctl enable horizon


    Horizon
        End-point: horizondb.cxsdezvxbyit.us-east-2.rds.amazonaws.com
        Port: 5432
        DB instance identifier: horizondb
        Master user name: horizon
        Master password: Welc0me123
        DB name: horizon
