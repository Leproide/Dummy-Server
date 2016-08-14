Imports System
Imports System.IO
Imports System.Net
Imports System.Net.Sockets
Imports System.Text
Imports Microsoft.VisualBasic


Module Module1


Class MyTcpListener

        Public Shared Sub Main()
            Console.Title = "Leprechaun's DummyServer"
            Dim logfile As String
            Try

                ' Set the TcpListener port
                Dim sr As New StreamReader("porta.txt")
                Dim line As String
                line = sr.ReadToEnd()
                Dim port As String = line

                Dim server As New TcpListener(IPAddress.Any, port)

                ' Start listening for client requests.
                server.Start()

                ' Buffer for reading data
                Dim bytes(1024) As [Byte]
                Dim data As [String] = Nothing

                ' Enter the listening loop.
                While True
                    logfile = Now.Day & "-" & Now.Month & "-" & Now.Year & ".txt"
                    Console.Write("Waiting for a connection... ")

                    ' Perform a blocking call to accept requests.
                    ' You could also user server.AcceptSocket() here.
                    Dim client As TcpClient = server.AcceptTcpClient()
                    Dim ClientRemoteIP As String = client.Client.RemoteEndPoint.ToString.Remove(client.Client.RemoteEndPoint.ToString.IndexOf(":"))
                    Console.WriteLine(Environment.NewLine)
                    Console.WriteLine("-------------------------------------------------------")
                    Console.WriteLine("Connected IP: " + ClientRemoteIP)
                    Using writer As StreamWriter = New StreamWriter(logfile, True)
                        writer.WriteLine("---------------------------------------" + vbCrLf + vbCrLf + DateTime.Now + " - Connected IP: " + ClientRemoteIP + " - Port: " + port)
                    End Using
                    Console.Beep()

                    data = Nothing

                    ' Get a stream object for reading and writing
                    Dim stream As NetworkStream = client.GetStream()

                    Dim i As Int32

                    ' Loop to receive all the data sent by the client.
                    i = stream.Read(bytes, 0, bytes.Length)
                    While (i <> 0)
                        ' Translate data bytes to a ASCII string.
                        data = System.Text.Encoding.ASCII.GetString(bytes, 0, i)
                        Console.WriteLine([String].Format("Received: {0}", data))

                        ' Process the data sent by the client.
                        data = data.ToUpper()

                        Dim msg As [Byte]() = System.Text.Encoding.ASCII.GetBytes(data)

                        ' Send back a response.
                        stream.Write(msg, 0, msg.Length)
                        Console.WriteLine([String].Format("Sent: {0}", data))

                        Using writer As StreamWriter = New StreamWriter(logfile, True)
                            writer.WriteLine(data + vbCrLf + vbCrLf)
                        End Using

                        i = stream.Read(bytes, 0, bytes.Length)

                    End While

                    ' Shutdown and end connection
                    client.Close()
                End While
            Catch e As SocketException
                Console.WriteLine("SocketException: {0}", e)
            End Try

            Console.WriteLine(ControlChars.Cr + "Hit enter to continue...")
            Console.Read()
        End Sub 'Main
    End Class 'MyTcpListener 

End Module
