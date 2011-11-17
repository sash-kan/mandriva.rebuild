LC_ALL = C
mirror = /mnt/mirrors/mandriva/official/2011
user = mandriva
v = 0

archs = i586 x86_64
srpms = $(shell cat srpms.list)

cachedir = cache
chrootdir = chroot
reportdir = report
failedlogdir = failedlogs
reportpackages = $(foreach p,$(addprefix $(reportdir)/,$(srpms)),$(foreach a,$(archs),$(p).$(a)))

srpmsincache = $(addprefix $(cachedir)/,$(srpms))

repos1 = main contrib non-free
repos2 = release updates
repos3 = $(foreach r1,$(repos1),$(foreach r2,$(repos2),$(r1)/$(r2)))
repos  = $(foreach r,$(repos3),$(foreach p,SRPMS $(archat)/media,$(p)/$(r)))

percent = %
archat = $(strip $(foreach a,$(archs),$(findstring $(a),$@)))
crat = $(chrootdir)/$(archat)
pkgat = $(basename $(notdir $@))
ifeq ($(v),0)
	output = &>/dev/null
	at = @
else
ifeq ($(v),1)
	output = &>/dev/null
	at =
else
	output =
	at =
endif
endif

all: srpms.list $(cachedir) $(reportdir) $(chrootdir) $(reportpackages)

$(reportpackages):
	@echo
	$(at)echo report: $@
	$(at)echo umount
	$(at)[ -f .sysmounted.$(archat) ] && sudo umount -lf $(crat)/sys && rm .sysmounted.$(archat); :
	$(at)[ -f .procmounted.$(archat) ] && sudo umount -lf $(crat)/proc && rm .procmounted.$(archat); :
	$(at) echo rm old chroot
	$(at)-sudo rm -rf $(crat)
	$(at)echo add medias
	$(at)for i in $(repos); do \
		sudo urpmi.addmedia --urpmi-root $(crat) $$(echo $$i | sed 's!/!_!g') \
		$(mirror)/$$i; done $(output)
	$(at)echo base system
	$(at)sudo urpmi  --no-suggests --excludedocs --no-verify-rpm --auto --root $(crat) \
	--urpmi-root $(crat) shadow-utils rpm tar basesystem-minimal rpm-build \
	rpm-mandriva-setup urpmi rsync bzip2 shadow-utils locales-en $(output)
	$(at)echo check cache
	$(at)if [ ! -f $(cachedir)/$(pkgat) ] ; then \
		p=$$(urpmq --urpmi-root $(crat) --sources \
			$(basename $(pkgat)) 2>.urpmq.error); \
		if [ -z "$$p" ] ; then \
			touch $(cachedir)/$(pkgat); \
			for i in $(archs); do cat .urpmq.error > $@; done; \
		else \
			cp $$p $(cachedir)/$(pkgat); \
		fi; \
		rm .urpmq.error; \
	fi
	$(at)echo proceed if size of file in cache greater than zero
	$(at)if [ -s $(cachedir)/$(pkgat) ] ; then \
		echo install buildreqs; \
		sudo urpmi --noclean --no-suggests --excludedocs --no-verify-rpm --auto \
			--root $(crat) --urpmi-root $(crat) --buildrequires \
			$(cachedir)/$(pkgat) $(output); \
		echo mount sys+proc; \
		[ ! -f .procmounted.$(archat) ] && sudo mount -o bind /proc $(crat)/proc \
			&& touch .procmounted.$(archat); \
		[ ! -f .sysmounted.$(archat) ] && sudo mount -o bind /sys $(crat)/sys \
			&& touch .sysmounted.$(archat); \
		echo create user $(user); \
		sudo chroot $(crat) adduser $(user); \
		echo install pkg; \
		sudo cp $(cachedir)/$(pkgat) $(crat)/tmp; \
		sudo chroot $(crat) su - $(user) -c '/bin/rpm --nodeps -i /tmp/$(pkgat) $(output)'; \
		echo build pkg; \
		sudo chroot $(crat) su - $(user) -c '/usr/bin/rpmbuild -ba rpmbuild/SPECS/*.spec \
			--target=$(archat)' | tee .log $(output) 2>&1; \
		echo check for errors; \
		if [ ! -f $(crat)/home/$(user)/rpmbuild/SRPMS/$(pkgat) ] ; then \
			cp .log $(failedlogdir)/$(notdir $@).log; \
			echo "no srpm" > $@; \
		else \
			echo ok > $@; \
		fi; \
		rm .log; \
	fi
	$(at)touch $@



clearall: delete.cachedir delete.reportdir delete.chrootdir delete.failedlogdir 

$(cachedir) $(reportdir) $(chrootdir):
	$(at)mkdir -p $@

delete.%:
	$(at)-sudo rm -rf $($*)

delete.chrootdir:
	$(at)-for i in $(archs); do \
		[ -f .sysmounted.$$i ] && sudo umount -lf $(chrootdir)/$$i/sys; \
		[ -f .procmounted.$$i ] && sudo umount -lf $(chrootdir)/$$i/proc; \
		rm .sysmounted.$$i .procmounted.$$i; \
		sudo rm -rf $(chrootdir)/$$i; \
	done
	$(at)-rmdir $(chrootdir)

