internal async Task ReceberMensagensAsync (CancellationToken stoppingToken) {
            bool pararBuscaMensagens = false;
            while (!stoppingToken.IsCancellationRequested) {
                try {
                    _logger.Information ("Recebendo mensagens em: {time}", DateTimeOffset.Now);
                    var retornoMq = _filaMqService.ReceberMensagem ();
 
                    if (retornoMq.Item1.Count > 0) {
                        // Mensagens Recebidas
                        _logger.Information ("Salvando mensagens em: {time}", DateTimeOffset.Now);
                    }
 
                    retornoMq.Item1.ToList ().ForEach (async msg => {
                        _logger.Information ("Mensagem recebida: '{0}'", msg.Value);
 
                        await _bancoDadosService.SalvarMensagemAsync (msg.Value);
 
                        await _bancoDadosService.SalvarMensagemAsync (msg.Value);
 
                        // Exluindo mensagens recebidas
                        var retornoLogsMq = _filaMqService.ExcluirMensagens ([msg.Key]);
                        retornoLogsMq.ForEach (msg => _logger.Information (msg));
                    });
                    GC.Collect ();
 
                    //Log receber mensagens
                    retornoMq.Item2.ForEach (msg => _logger.Information (msg));
                    pararBuscaMensagens = retornoMq.Item2.Any (msg =>
                        msg.Equals ("Não existem mensagens!", StringComparison.CurrentCultureIgnoreCase) ||
                        msg.Equals ("ReasonCode", StringComparison.CurrentCultureIgnoreCase)
                    );
 
                    GC.Collect ();
                } catch (Exception ex) {
                    _logger.Error ($"Erro - Msg:{ex.Message} - Trak: {ex.StackTrace}");
                    pararBuscaMensagens = true;
                }
 
                if (pararBuscaMensagens) {
                    _logger.Information ("Aguardando {0} minutos para próxima busca de mensagens", _tempoEsperaMinutos);
                    await Task.Delay (_tempoEsperaMilisegundos, stoppingToken);
                }
            }
        }# projeto_web
