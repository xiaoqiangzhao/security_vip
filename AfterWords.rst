
AfterWords
==========
For security subsystem , there are hundreds of patterns , and several vectors with so many IPs involved .And due to aggressive schedule, we don't have enough time to look deep into every pattern to see what actually they are doing or whether any coding style issue and False-Pass issue exists . But from those we did check the details, I'm afraid the total quality of those patterns derived from QM do need rigorous review .

And for those Procedure-Block Drivers provided by our SEC VIP, as mentioned, some of them may be necessary, and some of them are just workaround for quick setup . Coverage loss could be the penalty and perhaps we will need to analysis them for potential risk.

Another potential risk is about synchronization between i.MXQM and i.MXQXP. As we all know, our SEC VIP originates from the one of i.MXQM and is maintained on a separate vault. Since they are both immature and still in developing , we will need to sync for some common parts and make separate updates for design changes. It would be difficult to trace valuable update for every release of i.MXQM. Especially when we have already get most patterns passed , we will not look into i.MXQM for updates any more. If there are some issues within earlier releases and not informed by QM side, there may be some risk.
